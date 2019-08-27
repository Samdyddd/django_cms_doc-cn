.. _toolbar_how_to:

#########################
How to extend the Toolbar
如何扩展工具栏
#########################

django CMS工具栏提供了一个API，允许您在自己的代码中添加、删除和操作工具栏项。
它帮助您将django CMS的前端编辑模式集成到您的应用程序中，并为您的用户提供一个精简的编辑体验

..  seealso::

    * :ref:`Extending the Toolbar <toolbar_introduction>` in the tutorial
    * :ref:`Toolbar API reference <toolbar-api-reference>`


*********************************
Create a ``cms_toolbars.py`` file
*********************************

为了与工具栏API交互，您需要在自己的代码中创建一个:class:`~cms.toolbar_base.CMSToolbar`，并注册它。

这个类应该在应用程序的``cms_toolbars.py`` 文件中创建，当Django运行服务器启动时，它将自动被发现。

您还可以使用:setting:`CMS_TOOLBARS`来控制加载了哪些工具栏类。

..  admonition:: Use the high-level toolbar APIs 使用高级工具栏api

    你会在视图的请求中找到一个工具栏对象，你可能会想用它来做一些事情，比如:

    ..  code-block:: python

        toolbar = request.toolbar
        toolbar.add_modal_button('Do not touch', dangerous_button_url)

    \- 但是你不应该这样做，就像不建议你仅仅因为可以就把镊子插进插座里一样。.

    相反，您应该只使用``CMSToolbar`` 类和用于管理它的文档
    :ref:`documented APIs for managing it <toolbar-api-reference>`化api与工具栏进行交互。

    类似地，虽然有一个通用的:meth:`~cms.toolbar.items.ToolbarAPIMixin.add_item`方法，
    但是我们提供了处理特定项类型的高级方法，建议您使用这些方法。


**********************************************
Define and register a ``CMSToolbar`` sub-class
**********************************************

..  code-block:: python

    from cms.toolbar_base import CMSToolbar
    from cms.toolbar_pool import toolbar_pool

    class MyToolbarClass(CMSToolbar):
        [...]

    toolbar_pool.register(MyToolbarClass)

The ``cms.toolbar_pool.ToolbarPool.register`` 寄存器方法也可以用作装饰器:

..  code-block:: python
    :emphasize-lines: 1

    @toolbar_pool.register
    class MyToolbarClass(CMSToolbar):
        [...]


********************
Populate the toolbar
填充工具栏
********************

有两种方法可以控制django CMS工具栏中出现的内容:

* ``populate()``,在呈现页面的其余部分之前调用
* ``post_template_populate()``, 页面模板呈现后调用哪个

后一种方法允许您基于页面的内容来管理工具栏，例如插件或占位符的状态，
但是除非您需要这样做，否则您应该选择更简单的``populate()``方法。

..  code-block:: python
    :emphasize-lines: 3-5

    class MyToolbar(CMSToolbar):

        def populate(self):

            # add items to the toolbar

现在您必须确定工具栏中将显示哪些项目。这些可以包括:

* :ref:`menus <create-toolbar-menu>`
* :ref:`buttons <create-toolbar-button>` and button lists
* various other toolbar items


Add links and buttons to the toolbar
向工具栏添加链接和按钮
====================================

您可以使用各种``add_``方法将链接和按钮作为条目添加到菜单实例中。

====================== ============================================================= ===========================================================
Action                 Text link variant                                             Button variant
====================== ============================================================= ===========================================================
Open link              :meth:`~cms.toolbar.items.ToolbarAPIMixin.add_link_item`      :meth:`~cms.toolbar.toolbar.CMSToolbar.add_button`
Open link in sideframe :meth:`~cms.toolbar.items.ToolbarAPIMixin.add_sideframe_item` :meth:`~cms.toolbar.toolbar.CMSToolbar.add_sideframe_button`
Open link in modal     :meth:`~cms.toolbar.items.ToolbarAPIMixin.add_modal_item`     :meth:`~cms.toolbar.toolbar.CMSToolbar.add_modal_button`
POST action            :meth:`~cms.toolbar.items.ToolbarAPIMixin.add_ajax_item`
====================== ============================================================= ===========================================================

使用其中任何一种的基本形式是:

..  code-block:: python

    def populate(self):

        self.toolbar.add_link_item( # or add_button(), add_modal_item(), etc
            name='A link',
            url=url
            )

注意，尽管这些工具栏项的方法中可能包含各种位置参数，但是我们强烈建议使用上面提到的命名参数。
这将有助于确保您自己的工具栏类和方法在升级后仍然存在。
有关每个方法签名的详细信息，请参阅上表中链接到的参考文档。


Opening a URL in an iframe
在iframe中打开url
--------------------------

一种常见的情况是提供一个URL，该URL在同一页面的侧框或模式对话框中打开。
在站点菜单中，在侧框中打开Django admin就是一个很好的例子。侧框和模态都是HTML iframe。

A typical use for a sideframe is to display an admin list (similar to that used in the
:ref:`tutorial example <add-nodes-to-polls-menu>`):
侧边框的一个典型用法是显示一个管理列表(与教程示例中使用的类似):

..  code-block:: python
    :emphasize-lines: 1, 8-11

    from cms.utils.urlutils import admin_reverse
    [...]

    class PollToolbar(CMSToolbar):

        def populate(self):

            self.toolbar.add_sideframe_item(
                name='Poll list',
                url=admin_reverse('polls_poll_changelist')
                )

模态项的一个典型用法是显示模型实例的admin:

..  code-block:: python

        self.toolbar.add_modal_item(name='Add new poll', url=admin_reverse('polls_poll_add'))

但是，您不受限于这些示例，您可以在模态或侧框中打开任何合适的资源。
注意，协议可能需要匹配，并且请求的资源必须允许匹配。


..  _create-toolbar-button:

Adding buttons to the toolbar
向工具栏添加按钮
-----------------------------

A button is a sub-class of :class:`cms.toolbar.items.Button`

按钮也可以添加到列表中——:class:`~cms.toolbar.items.ButtonList`是一组可视化链接的按钮。

..  code-block:: python
    :emphasize-lines: 3-5

    def populate(self):

        button_list = self.toolbar.add_button_list()
        button_list.add_button(name='Button 1', url=url_1)
        button_list.add_button(name='Button 2', url=url_2)


..  _create-toolbar-menu:

Create a toolbar menu
创建toolbar 菜单
=====================

上面描述的文本链接项也可以作为节点添加到工具栏中的菜单中。

菜单是 :class:`cms.toolbar.items.Menu`的一个实例。 在 ``CMSToolbar`` 子类中,
可以使用:meth:`~cms.toolbar.toolbar.CMSToolbar.get_or_create_menu`在``populate()`` or ``post_template_populate()``方法中创建菜单，或者
标识已存在的菜单(例如，为了向其中添加新项).

..  code-block:: python

    def populate(self):
        menu = self.toolbar.get_or_create_menu(
            key='polls_cms_integration',
            verbose_name='Polls'
            )


键是唯一的菜单标识符;``verbose_name``将显示在菜单中。如果您知道一个菜单已经存在，您可以使用``get_menu()``获取它。

..  note::

    建议将密钥命名为应用程序名称。否则，另一个应用程序可能会意外地干扰您的菜单

一旦有了菜单，就可以像添加工具栏一样向其中添加项目。例如:

..  code-block:: python
    :emphasize-lines: 4-7

    def populate(self):
        menu = [...]

        menu.add_sideframe_item(
            name='Poll list',
            url=admin_reverse('polls_poll_changelist')
        )


To add a menu divider
要添加菜单分隔符
---------------------

 

:meth:`~cms.toolbar.items.SubMenu.add_break`将在菜单列表中放置一个
:class:`~cms.toolbar.items.Break`(可视分隔符)，以允许对项进行分组。例如:

..  code-block:: python

    menu.add_break(identifier='settings_section')


To add a sub-menu
-----------------

子菜单是属于另一个菜单的 ``Menu``:

..  code-block:: python
    :emphasize-lines: 4-7

    def populate(self):
        menu = [...]

        submenu = menu.get_or_create_menu(
            key='sub_menu_key',
            verbose_name='My sub-menu'
            )

然后，可以使用与上面示例相同的方法向子菜单添加项。注意，子菜单是子菜单的一个实例，它本身可能没有更多的子菜单。


.. _finding_toolbar_items:

******************************
Finding existing toolbar items
查找现有工具栏项
******************************

``get_or_create_menu()`` and ``get_menu()``
===========================================

存在许多方法和有用的常量来获取和操作现有的工具栏项。例如，要查找(使用get_menu())并重命名站点菜单:

..  code-block:: python

    from cms.cms_toolbars import ADMIN_MENU_IDENTIFIER

    class ManipulativeToolbar(CMSToolbar):

        def populate(self):

            admin_menu = self.toolbar.get_menu(ADMIN_MENU_IDENTIFIER)

            admin_menu.name = "Site"

``get_or_create_menu()`` 也能很好地找到相同的菜单，并且具有以下优点:

* 它可以更新项目的属性本身
  (``self.toolbar.get_or_create_menu(ADMIN_MENU_IDENTIFIER, 'Site')``)
* 如果项目不存在，它将创建它，而不是引发错误。


``find_items()`` and ``find_first()``
=====================================

Search for items by their type:

..  code-block:: python

    def populate(self):

        self.toolbar.find_items(item_type=LinkItem)

将在工具栏中找到所有链接项(但不是在工具栏中的菜单中——它不会在工具栏中搜索其他项以找到它们自己的项)。

:meth:`~cms.toolbar.items.ToolbarAPIMixin.find_items` 返回
:class:`~cms.toolbar.items.ItemSearchResult` 对象列表;
:meth:`~cms.toolbar.items.ToolbarAPIMixin.find_first` 返回列表中的第一个对象. 它们具有相似的行为，因此这里的示例只使用 ``find_items()`` 。

The ``item_type`` 参数总是必需的，但是您可以使用它们的其他属性来优化搜索，例如:

    self.toolbar.find_items(Menu, disabled=True))

注意，您也可以使用这两个方法来搜索菜单和子菜单类中的项。

.. _toolbar_control_item_position:

********************************************
Control the position of items in the toolbar
控制工具栏中项的位置
********************************************

向工具栏添加菜单项的方法采用可选的位置参数，该参数可用于控制项将插入的位置。

默认情况下(``position=None``)，项将插入到与层次结构相同级别的现有项之后
(一个新子菜单将成为菜单的最后一个子菜单，一个新菜单将成为工具栏中的最后一个菜单，依此类推)。

0的位置将在所有其他项之前插入该项。

如果已经有对象，也可以将其用作引用。例如:

..  code-block:: python

    def populate(self):

        link = self.toolbar.add_link_item('Link', url=link_url)
        self.toolbar.add_button('Button', url=button_url, position=link)

将在链接项之前添加新按钮。

最后，您可以使用:class:`~cms.toolbar.items.ItemSearchResult`作为位置:

..  code-block:: python

    def populate(self):

        self.toolbar.add_link_item('Link', url=link_url)

        link = self.toolbar.find_first(LinkItem)

        self.toolbar.add_button('Button', url=button_url, position=link)

由于``ItemSearchResult``可以转换为整数，您甚至可以这样做:

    self.toolbar.add_button('Button', url=button_url, position=link+1)


****************************************
Control how and when the toolbar appears
控制工具栏出现的方式和时间
****************************************

默认情况下，当用户``is_staff``时，您的:class:`~cms.toolbar_base.CMSToolbar`子类将在每个页面的工具栏中处于活动状态(即调用它的填充方法)。
然而，有时候``CMSToolbar``子类只应该在访问与特定应用程序关联的页面时填充工具栏。

``CMSToolbar`` 子类有一个有用的属性，可以帮助确定是否应该激活工具栏。
当包含工具栏类的应用程序与处理请求的应用程序匹配时，is_current_app为``True``。

这让你可以选择性地激活它，例如:

..  code-block:: python
    :emphasize-lines: 3-4

    def populate(self):

        if not self.is_current_app:
            return

        [...]

如果你的工具栏类在另一个应用程序中，而不是你希望它被激活的应用程序中，
你可以列出当你创建这个类时它应该支持的任何应用程序:

..  code-block:: python

    supported_apps = ['some_app']

``supported_apps`` 是应用程序点状路径 (e.g: ``supported_apps =
('whatever.path.app', 'another.path.app')``.

属性``app_path``将包含处理当前请求的应用程序的名称——
如果``app_path``位于``supported_apps``中，则is_current_app为``True``。

*****************************
Modifying an existing toolbar
修改现有工具栏
*****************************

如果您需要修改现有的工具栏(例如更改属性或方法的行为)，您可以通过创建实现所需更改的工具栏子类，
并注册它而不是原始工具栏来实现。

可以使用``toolbar_pool.unregister()``注销原始文件，如下面的示例所示。
另外，如果您最初使用CMS_TOOLBARS调用工具栏类，则需要修改它以引用新的工具栏。

例如，我们注销原始文件，而注册我们自己的文件:


    from cms.toolbar_pool import toolbar_pool
    from third_party_app.cms_toolbar import ThirdPartyToolbar

    @toolbar_pool.register
    class MyBarToolbar(ThirdPartyToolbar):
        [...]

    toolbar_pool.unregister(ThirdPartyToolbar)


.. _url_changes:

**********************************
Detecting URL changes to an object
检测对象的URL更改
**********************************

如果您想查看模型的对象创建或编辑，并在添加或更改模型后重定向，请在工具栏中添加watch_models属性。

Example::

    class PollToolbar(CMSToolbar):

        watch_models = [Poll]

        def populate(self):
            ...

添加此选项后，通过侧框或模态窗口对Poll实例的每次更改都将根据工具栏状态，触发重定向到已编辑的Poll实例的URL:

* 在draft模式中返回``get_draft_url()``(如果不存在``get_absolute_url()``，则返回``get_absolute_url()``)
* 在活动模式下，方法存在，返回``get_public_url()``。


********
Frontend
********

如果您需要与工具栏交互，或者在站点的前端代码中解释它，它将提供CSS和JavaScript挂钩供您使用。

它将添加各种类到页面的<html>元素:

* ``cms-ready``, 当工具栏准备好时
* ``cms-toolbar-expanded``, 当工具栏完全展开时
* ``cms-toolbar-expanding`` and ``cms-toolbar-collapsing``在工具栏动画期间.

工具栏还会在文档上触发一个名为cms-ready的JavaScript事件。你可以用jQuery听这个事件:

    CMS.$(document).on('cms-ready', function () { ... });
