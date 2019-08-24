.. _toolbar_introduction:

#####################
Extending the toolbar
#####################

django CMS允许您控制工具栏中出现的内容。
这允许您将应用程序集成到django CMS的前端编辑模式中，并为用户提供一种流线型的编辑体验。

在本教程的这一部分中，我们将向工具栏添加一个新的民意调查菜单。

*********************************
Add a basic ``PollToolbar`` class
*********************************

我们将使用 ``cms.toolbar_base.CMSToolbar`` 向工具栏添加各种控件。CMSToolbar子类。


Add a menu to the toolbar
=========================

首先添加一个新文件cms_toolbar.py，然后创建CMSToolbar类:

..  code-block:: python

    from cms.toolbar_base import CMSToolbar
    from cms.toolbar_pool import toolbar_pool
    from polls.models import Poll


    class PollToolbar(CMSToolbar):

        def populate(self):
            self.toolbar.get_or_create_menu(
                'polls_cms_integration-polls',  # a unique key for this menu
                'Polls',                        # the text that should appear in the menu
                )


    # register the toolbar
    toolbar_pool.register(PollToolbar)


..  note::

    不要忘记重新启动runserver以识别新的cms_tools.py文件。

现在，你会发现，在网站的每一页，一个新的项目在工具栏:

.. image:: /introduction/images/toolbar-polls.png
   :alt: The Polls menu in the toolbars
   :width: 630

生成工具栏时调用populate()方法。
在其中，我们使用get_or_create_menu()向工具栏添加一个poll项。


.. _add-nodes-to-polls-menu:

Add nodes to the *Polls* menu
-----------------------------

到目前为止，民意调查菜单是空的。我们可以扩展populate()来添加一些项。
get_or_create_menu返回一个我们可以操作的菜单，因此让我们更改populate()方法来添加一个项，
它允许我们使用add_sideframe_item()查看侧框中的完整轮询列表。

..  code-block:: python
    :emphasize-lines: 1, 8, 10-13

    from cms.utils.urlutils import admin_reverse
    [...]


    class PollToolbar(CMSToolbar):

        def populate(self):
            menu = self.toolbar.get_or_create_menu('polls_cms_integration-polls', 'Polls')

            menu.add_sideframe_item(
                name='Poll list',                              # name of the new menu item
                url=admin_reverse('polls_poll_changelist'),    # the URL it should open with
            )

刷新页面以加载更改后，现在可以直接从菜单中看到投票列表。

另外一个有用的选项是创建新的民意调查。我们将为此使用一个模态窗口，用add_modal_item()调用。
将新代码添加到populate()方法的末尾:

..  code-block:: python
    :emphasize-lines: 6-9

    class PollToolbar(CMSToolbar):

        def populate(self):
            [...]

            menu.add_modal_item(
                name='Add a new poll',                # name of the new menu item
                url=admin_reverse('polls_poll_add'),  # the URL it should open with
            )


Add buttons to the toolbar
==========================

除了菜单，您还可以以非常类似的方式向工具栏添加按钮。
重写populate()方法，注意此代码的结构与添加菜单的结构匹配程度。

..  code-block:: python
    :emphasize-lines: 3-13

    def populate(self):

        buttonlist = self.toolbar.add_button_list()

        buttonlist.add_sideframe_button(
            name='Poll list',
            url=admin_reverse('polls_poll_changelist'),
        )

        buttonlist.add_modal_button
            name='Add a new poll',
            url=admin_reverse('polls_poll_add'),
        )


*******************
Further refinements
*******************

用于投票的按钮和菜单出现在站点的工具栏中。将其限制在实际相关的页面上是有用的。

要添加的第一件事是在populate()方法的开始处添加一个测试:

..  code-block:: python
    :emphasize-lines: 3-4

        def populate(self):

            if not self.is_current_app:
                return

            [...]

``is_current_app`` 标志告诉我们，处理这个视图的函数(例如，轮询列表)是否属于负责这个工具栏菜单的应用程序。

通常，这可以自动检测到，但在本例中，视图属于poll应用程序，而工具栏菜单属于polls_cms_integration。
因此，我们需要显式地告诉PollToolbar类它实际上与poll应用程序相关联:

..  code-block:: python
    :emphasize-lines: 3

    class PollToolbar(CMSToolbar):

        supported_apps = ['polls']

 buttons/menu 将只出现在相关页面中。


********************************
The complete ``cms_toolbars.py``
********************************

为了完整起见，下面是完整的例子:

..  code-block:: python

    from cms.utils.urlutils import admin_reverse
    from cms.toolbar_base import CMSToolbar
    from cms.toolbar_pool import toolbar_pool
    from polls.models import Poll


    class PollToolbar(CMSToolbar):
        supported_apps = ['polls']

        def populate(self):

            if not self.is_current_app:
                return

            menu = self.toolbar.get_or_create_menu('polls_cms_integration-polls', 'Polls')

            menu.add_sideframe_item(
                name='Poll list',
                url=admin_reverse('polls_poll_changelist'),
            )

            menu.add_modal_item(
                name=('Add a new poll'),
                url=admin_reverse('polls_poll_add'),
            )

            buttonlist = self.toolbar.add_button_list()

            buttonlist.add_sideframe_button(
                name='Poll list',
                url=admin_reverse('polls_poll_changelist'),
            )

            buttonlist.add_modal_button(
                name='Add a new poll',
                url=admin_reverse('polls_poll_add'),
            )

    toolbar_pool.register(PollToolbar)  # register the toolbar

这只是一个基本的例子，django CMS工具栏类还有很多——请参阅如何扩展工具栏以获得更多信息- see
:ref:`toolbar_how_to` 。
