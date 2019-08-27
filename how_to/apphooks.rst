.. _apphooks_how_to:

######################
How to create apphooks
######################

apphook允许您将Django应用程序附加到页面上。
例如，您可能希望将一个新闻应用程序集成到django CMS中。在这种情况下，
您可以创建一个不包含任何内容的普通django CMS页面，
并将新闻应用程序附加到页面上;新闻应用程序的内容将在页面的URL中交付。

该URL路径中的所有URL都将传递给附加应用程序的URL配置。

教程 :ref:`Tutorials <tutorials>` 部分包含了开始使用apphooks的基本指南。本文档假定您对CMS比较熟悉。

******************************
The basics of apphook creation
apphook创建的基础知识
******************************

要创建apphook，请在应用程序中创建``cms_apps.py``文件。
该文件需要包含一个``CMSApp``子类。例如:

    from cms.app_base import CMSApp
    from cms.apphook_pool import apphook_pool

    @apphook_pool.register
    class MyApphook(CMSApp):
        app_name = "myapp"  # must match the application namespace
        name = "My Apphook"

        def get_urls(self, page=None, language=None, **kwargs):
            return ["myapp.urls"] # replace this with the path to your application's URLs module

.. versionchanged:: 3.3
    ``CMSApp.get_urls()`` replaces ``CMSApp.urls``. ``urls`` was removed
    in version 3.5.


Apphooks for namespaced applications
用于命名空间应用程序的apphook
====================================

应用程序应该使用:ref:`namespaced URLs <django:topics-http-defining-url-namespaces>`。

在上面的示例中，应用程序使用``myapp``名称空间。您的``CMSApp``子类必须在``app_name``属性中反映应用程序的名称空间。

应用程序可以通过在其``urls.py``中提供``app_name``来指定名称空间。或其文档可能会建议您在包含它的URLs时这样做:

..  code-block:: python

    url(r'^myapp/', include('myapp.urls', app_name='myapp'))

如果您未能做到这一点,那么应用程序中使用表单 
``{% url 'myapp:index' %}`` 调用url的任何模板或调用(for example) ``reverse('myapp:index')`` 的视图都会抛出``NoReverseMatch``错误



Apphooks for non-namespaced applications
用于非名称空间应用程序的apphook
----------------------------------------

如果您正在为第三方应用程序编写apphooks，您可能会发现其中一个实际上没有用于url的应用程序名称空间。
这样的应用程序很容易陷入名称空间冲突，并且不代表良好的实践。

但是，如果您遇到这样的应用程序，您自己的apphook将需要放弃``app_name``属性。

注意，与没有``app_name``属性的apphooks不同，一次只能附加到一个页面;
第二次尝试应用它们将导致错误。这些apphook只能存在一个实例。

See :ref:`multi_apphook`实例的更多信息, 请参见多次附加应用程序。


Returning apphook URLs manually
手动返回apphook urls
===============================

而不是在另一个文件``myapp/urls.py``中定义URL模式。，也可以手动返回它们，例如，
如果需要覆盖提供的集合。一个例子:
..  code-block:: python

    from django.conf.urls import url
    from myapp.views import SomeListView, SomeDetailView

    class MyApphook(CMSApp):
        # ...
        def get_urls(self, page=None, language=None, **kwargs):
            return [
                url(r'^$', SomeListView.as_view()),
                url(r'^(?P<slug>[\w-]+)/?$', SomeDetailView.as_view()),
                ]

但是，将它们保存在应用程序的``urls.py``中要整洁得多。它们可以很容易地重用。

.. _reloading_apphooks:

Loading new and re-configured apphooks
加载新的和重新配置的apphook
======================================

某些与apphook相关的更改需要重新启动服务器才能加载。

Whenever you:

* 添加或删除一个apphook
* 更改包含apphook的页的slug或具有apphook子代的页的slug

URL缓存必须重新加载。

如果你有安装 :ref:`ApphookReloadMiddleware`, 服务器将通过自动重新初始化URL模式来为您完成。

否则，您将需要手动重启服务器。


****************
Using an apphook
****************

安装并加载apphook之后，现在就可以从高级设置中选择连接到该页面的应用程序。

.. note::

    apphook在它所属的页面发布之前实际上什么都不会做。请注意，这也意味着所有父页面也必须被发布。

apphook将apphooked应用程序的所有url附加到页面;它的根URL将是页面自己的URL，
任何较低级别的URL都将位于相同的URL路径上。

So, given an application with the ``urls.py`` for the views ``index_view`` and ``archive_view``::

    urlpatterns = [
        url(r'^$', index_view),
        url(r'^archive/$', archive_view),
    ]

连接到URL路径为 ``/hello/world/``, 视图将显示如下:

* ``index_view`` at ``/hello/world/``
* ``archive_view`` at ``/hello/world/archive/``


Sub-pages of an apphooked page
apphooked页面的子页面
==============================

..  important::

    不要将子页面添加到带有apphook的页面。
    apphook会“吞噬”页面下方的所有url，并将它们传递给附加的应用程序。
    如果您有apphooked页面的任何子页面，django CMS将无法可靠地为它们提供服务。


*****************
Managing apphooks
*****************

Uninstalling an apphook with applied instances
卸载带有应用实例的apphook
==============================================

如果你从你的系统中删除一个apphook类(实际上是卸载它)，它仍然有实例应用于页面，django CMS会尽可能优雅地处理这个问题:

* 受影响的网页仍然保存应用apphook的记录;如果apphook类随后被恢复，它将像以前一样工作。
* 页面列表将在适当的地方显示apphook指示器。
* 否则，该页面将表现为一个正常的django CMS页面，并以通常的方式显示它的占位符。
* 如果您保存页面的高级设置，apphook将被删除。


Management commands
管理命令
===================

您可以使用CMS管理命令``uninstall apphooks``清除卸载的apphook实例。例如:
    manage.py cms uninstall apphooks MyApphook MyOtherApphook

您可以使用cms列表获得已安装apphooks的列表;在这种情况下:

    manage.py cms list apphooks

更多信息 :ref:`Management commands reference <management_commands>` 。


.. _apphook_menus:

************************
Adding menus to apphooks
向apphooks添加菜单
************************

通常，建议允许用户控制是否将菜单附加到页面上(有关这些菜单的更多信息，请参见附加菜单)。
然而，如果需要，apphook可以自动完成此操作。它的行为就像使用高级设置将菜单附加到页面一样)。

可以使用``get_menus()``方法将菜单添加到apphook中。基于上述例子:
    # [...]
    from myapp.cms_menus import MyAppMenu

    class MyApphook(CMSApp):
        # [...]
        def get_menus(self, page=None, language=None, **kwargs):
            return [MyAppMenu]

.. versionchanged:: 3.3
    ``CMSApp.get_menus()`` replaces ``CMSApp.menus``. The ``menus`` 属性现在已被弃用，并在version3.5版本中被删除。


 ``get_menus()``方法中返回的菜单需要在其``get_nodes()``方法中返回节点列表。
``get_nodes()`` methods. :ref:`integration_attach_menus` 附加菜单包含关于创建生成节点的菜单类的更多信息。

您可以返回多个菜单类;所有资料将附于同一页:

    def get_menus(self, page=None, language=None, **kwargs):
        return [MyAppMenu, CategoryMenu]


.. _apphook_permissions:

********************************
Managing permissions on apphooks
管理apphooks上的权限
********************************

默认情况下，apphook表示的内容具有与其分配给的页面相同的权限集。
因此，例如，如果一个页面需要用户登录，那么附加的apphook及其所有url将具有相同的要求。

若要禁用此行为，请在apphook上设置 ``permissions = False``::

    class MyApphook(CMSApp):
        [...]
        permissions = False

如果你仍然想让你的一些视图使用CMS的权限检查，你可以通过装饰器启用它们，, ``cms.utils.decorators.cms_perms``

Here is a simple example::

    from cms.utils.decorators import cms_perms

    @cms_perms
    def my_view(request, **kw):
        ...

如果您在您的应用程序中进行自己的权限检查，那么使用apphook的``exclude_permissions``属性:

    class MyApphook(CMSApp):
        [...]
        permissions = True
        exclude_permissions = ["some_nested_app"]

在哪里提供有问题的应用程序名称


***********************************************
Automatically restart server on apphook changes
apphook更改时自动重启服务器
***********************************************

如上所述，当你:

* add or remove an apphook
* change the slug of a page containing an apphook 更改包含apphook的页面的段塞
* 用带有apphook的后代来悬挂一页的粗体

服务器将重新加载其URL缓存。它通过侦听信号 ``cms.signals.urls_need_reloading``来实现这一点。

.. warning::

    这个信号本身没有任何作用。为了自动重启服务器，您需要在项目中实现逻辑，每当触发此信号时，都会执行该逻辑。
    因为部署Django应用程序的方法有很多，所以我们不可能提供一个通用的解决方案来解决这个问题。
    信号是在请求之后发出的——例如，在保存页面设置时。如果您通过API更改apphook的设置，则信号直到后续请求才会触发。


**************************************
Apphooks and placeholder template tags
Apphooks和占位符模板标记
**************************************

重要的是要理解，虽然apphooked应用程序完全接管了该位置的CMS页面，这取决于应用程序的模板如何扩展其他模板，
但django  CMS ``{% placeholder %}`` 模板标记可能会被调用，但不会工作。

``{% static_placeholder %}`` 标记不是特定于页面的，将正常工作。
