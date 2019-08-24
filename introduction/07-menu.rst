#############################
Extending the navigation menu
#############################

您可能已经注意到，虽然我们的民意调查应用程序已经集成到CMS中，带有插件、工具栏菜单项等，
但是站点的导航菜单仍然只由django CMS页面决定。

我们可以挂接到django CMS菜单系统中，将我们自己的节点添加到导航菜单中。


**************************
Create the navigation menu
**************************

我们使用CMSAttachMenu子类创建菜单，并使用get_nodes()方法添加节点。

为此，我们需要在应用程序中使用一个名为 ``cms_menus.py`` 的文件， 在 ``polls_cms_integration/``添加``cms_menus.py``:

.. code-block:: python

    from django.urls import reverse
    from django.utils.translation import ugettext_lazy as _

    from cms.menu_bases import CMSAttachMenu
    from menus.base import NavigationNode
    from menus.menu_pool import menu_pool

    from polls.models import Poll


    class PollsMenu(CMSAttachMenu):
        name = _("Polls Menu")  # give the menu a name this is required.

        def get_nodes(self, request):
            """
            This method is used to build the menu tree.
            """
            nodes = []
            for poll in Poll.objects.all():
                node = NavigationNode(
                    title=poll.question,
                    url=reverse('polls:detail', args=(poll.pk,)),
                    id=poll.pk,  # unique id for this node within the menu
                )
                nodes.append(node)
            return nodes

    menu_pool.register_menu(PollsMenu)


What's happening here:

* 定义并注册了``PollsMenu`` class
* 我们给类一个name属性(将在admin中显示)
* 在它的get_nodes()方法中，我们构建并返回一个节点列表，whe
* 首先，我们获得所有的Poll对象
* ... and then create a ``NavigationNode`` object from each one
* ... and return a list of these ``NavigationNodes``，并返回这些导航节点的列表

在附加到页面之前，这个菜单类实际上什么都不会做。
在前面附加apphook的页面的高级设置中，从附加菜单选项列表中选择“polling Menu”，并再次保存。
(您可以将菜单添加到任何页面，但是将它添加到这个页面最有意义。)

.. image:: /introduction/images/attach-menu.png
   :alt: select the 'Polls Menu'
   :width: 400
   :align: center

如果您认为合适，可以强制apphook将菜单自动添加到页面中。
有关如何做到这一点的信息，请参见向apphooks添加菜单。

..  note::

    这里的重点是阐明基本原则。在这种实际情况下，请注意:

    * 如果要使用子页面，则需要改进菜单样式，使其更好地工作。
    * 因为民意调查页面列出了其中的所有民意调查，所以这并不是菜单中最实用的添加。
