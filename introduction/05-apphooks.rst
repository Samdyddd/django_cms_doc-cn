.. _apphooks_introduction:

########
Apphooks
########

现在，我们的Django民意调查应用程序被静态地连接到项目的``urls.py``中。
这没有关系，但是我们可以做得更多，将应用程序附加到django CMS页面。


*****************
Create an apphook
*****************

我们使用一个apphook来实现这一点，
这个apphook是使用CMSApp子类创建的，它告诉CMS如何包含该应用程序。


Create the apphook class
========================

Apphooks位于一个名为``cms_apps``的文件中,
因此，在``polls/CMS``集成应用程序中创建一个，即在polls_cms_integration中创建一个.

这是django CMS应用程序的一个非常基本的apphook示例:

.. code-block:: python

    from cms.app_base import CMSApp
    from cms.apphook_pool import apphook_pool


    @apphook_pool.register  # register the application
    class PollsApphook(CMSApp):
        app_name = "polls"
        name = "Polls Application"

        def get_urls(self, page=None, language=None, **kwargs):
            return ["polls.urls"]


In this ``PollsApphook`` class, we have done several key things:

* ``app_name`` 属性为系统提供了一种引用apphook的独特方式。
    从Django民意调查中可以看出，应用程序名称空间民意调查是硬编码到应用程序中的，因此这个属性也必须是民意调查。
* ``name`` is a human-readable name, 并将显示给管理用户。.
* ``get_urls()`` 方法是实际挂钩应用程序的方法，返回一个URL配置列表，
    该列表将在使用apphook的任何地方激活——在本例中，它将使用轮询中的``urls.py``。


Remove the old ``polls`` entry from the project's ``urls.py``
从项目的``urls.py``中删除旧的``polls``
=============================================================

你现在必须删除投票申请的项目::

    url(r'^polls/', include('polls.urls', namespace='polls'))

from your project's ``urls.py``.

它不仅不再是必需的，因为我们通过apphook到达投票，如果你把它放在那里，
它将与apphook的URL处理冲突。你会在日志中收到警告::

    URL namespace 'polls' isn't unique. You may not be able to reverse all URLs in this namespace.


Restart the runserver
=====================

**Restart the runserver**. 这是必要的，因为我们创建了一个包含Python代码的新文件，
直到服务器重新启动才会加载这些代码。您只需要在第一次创建新文件时执行此操作


.. _apply_apphook:

***************************
Apply the apphook to a page
***************************

现在我们需要创建一个新页面，并通过这个apphook将poll应用程序附加到它。

创建并保存一个新页面，然后发布它。

..  note:: 您的apphook在页面发布之前无法工作。

In its *Advanced settings* (from the toolbar, select *Page > Advanced settings...*) choose "Polls
Application" from the *Application* pop-up menu, 从应用程序弹出菜单中选择“polling Application”，并再次保存。

.. image:: /introduction/images/select-application.png
   :alt: select the 'Polls' application
   :width: 400
   :align: center

刷新该页面，您将发现poll应用程序现在可以直接从新的django CMS页面中获得。

..  important::

   不要将子页面添加到带有apphook的页面。
    apphook会“吞噬”页面下方的所有url，并将它们传递给附加的应用程序。
    如果您有apphooked页面的任何子页面，django CMS将无法可靠地为它们提供服务。
