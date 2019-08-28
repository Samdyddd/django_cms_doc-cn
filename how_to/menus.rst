#################################
How to customise navigation menus
如何定义导航菜单
#################################

在本文中，我们讨论了定制django CMS站点导航菜单的三种不同方法

1. :ref:`integration_menus`: 静态地扩展菜单项

2. :ref:`integration_attach_menus`: 将菜单附加到页面上。

3. :ref:`integration_modifiers`: 修改整个菜单树

.. _integration_menus:

*****
Menus
*****

创建 ``cms_menus.py`` 文件，如下所示::

    from menus.base import Menu, NavigationNode
    from menus.menu_pool import menu_pool
    from django.utils.translation import ugettext_lazy as _

    class TestMenu(Menu):

        def get_nodes(self, request):
            nodes = []
            n = NavigationNode(_('sample root page'), "/", 1)
            n2 = NavigationNode(_('sample settings page'), "/bye/", 2)
            n3 = NavigationNode(_('sample account page'), "/hello/", 3)
            n4 = NavigationNode(_('sample my profile page'), "/hello/world/", 4, 3)
            nodes.append(n)
            nodes.append(n2)
            nodes.append(n3)
            nodes.append(n4)
            return nodes

    menu_pool.register_menu(TestMenu)

.. note:: 在3.1版本之前，这个模块名为menu.py。
          请将现有模块更新为新的命名约定。3.5版将删除对旧名称的支持。

如果刷新一个页面，现在应该可以看到上面的菜单项。
``get_nodes``函数应该返回:class:`NavigationNode <menus.base.NavigationNode>`实例列表。
:class:`menus.base.NavigationNode`接受以下参数:

``title``
  菜单节点的文本

``url``
  菜单节点链接的URL

``id``
  此菜单的唯一id

``parent_id=None``
  如果这是另一个节点的子节点，则在这里提供父节点的id。

``parent_namespace=None``
  如果父节点不在此菜单中，则可以为其提供父名称空间。名称空间是类的名称。
  在上面的例子中是: ``TestMenu``

``attr=None``
  您可能希望在修饰符或模板中使用的附加属性的字典

``visible=True``
  此菜单项是否应可见

另外,每个:class:`menus.base.NavigationNode`提供了许多方法，
这些方法在:class:`NavigationNode <menus.base.NavigationNode>` API引用中有详细说明。

Customise menus at runtime
在运行时定制菜单
==========================

要根据依赖于请求的条件(例如:anonymous/logged(匿名/登录用户))调整菜单，可以使用`Navigation Modifiers`_导航修饰符，
也可以使用现有的修饰符。

：例如，可以添加由django CMS核心 ``AuthVisibility``修饰符识别的``{'visible_for_anonymous':
False}``/``{'visible_for_authenticated': False}``属性

Complete example::

    class UserMenu(Menu):
        def get_nodes(self, request):
                return [
                    NavigationNode(_("Profile"), reverse(profile), 1, attr={'visible_for_anonymous': False}),
                    NavigationNode(_("Log in"), reverse(login), 3, attr={'visible_for_authenticated': False}),
                    NavigationNode(_("Sign up"), reverse(logout), 4, attr={'visible_for_authenticated': False}),
                    NavigationNode(_("Log out"), reverse(logout), 2, attr={'visible_for_anonymous': False}),
                ]


.. _integration_attach_menus:

************
Attach Menus
附加菜单
************

扩展自:class:`menus.base.Menu`的类。菜单总是附加到根目录。但如果你想要菜单附加到CMS页面，你也可以这样做

您需要从:class:`~menus.base.Menu`扩展而不是从菜单扩展。
:class:`cms.menu_bases.CMSAttachMenu`，您需要定义一个名称。

我们将用上面的例子来做::

    from menus.base import NavigationNode
    from menus.menu_pool import menu_pool
    from django.utils.translation import ugettext_lazy as _
    from cms.menu_bases import CMSAttachMenu

    class TestMenu(CMSAttachMenu):

        name = _("test menu")

        def get_nodes(self, request):
            nodes = []
            n = NavigationNode(_('sample root page'), "/", 1)
            n2 = NavigationNode(_('sample settings page'), "/bye/", 2)
            n3 = NavigationNode(_('sample account page'), "/hello/", 3)
            n4 = NavigationNode(_('sample my profile page'), "/hello/world/", 4, 3)
            nodes.append(n)
            nodes.append(n2)
            nodes.append(n3)
            nodes.append(n4)
            return nodes

    menu_pool.register_menu(TestMenu)

现在，您可以将此菜单链接到“附加菜单下的页面设置”的“高级”选项卡中的页面。


.. _integration_modifiers:

********************
Navigation Modifiers
导航修饰符
********************

导航修饰符使应用程序可以访问导航菜单。
修饰符可以更改现有节点的属性或重新排列整个菜单。


Example use-cases
=================

一个简单的例子:您有一个新闻应用程序，它独立于django CMS发布页面。
但是，您希望将应用程序集成到站点的菜单结构中，以便在导航菜单中出现一个新闻节点。

在另一个示例中，您可能希望页面的特定属性在菜单模板中可用。
为了保持菜单节点的轻量级(这在拥有数千个页面的站点中可能很重要)，
它们只包含生成可用菜单所需的最小属性。

在这两种情况下,一个导航修改器解决方案——在第一种情况下,在适当的地方添加一个新节点,第二,添加一个新的属性——attr属性,
而不是直接NavigationNode,帮助避免利益冲突——菜单中的所有节点。

How it works
============

将修饰符放在应用程序的 ``cms_menus.py``.

要使修饰符可用，则需要在``menus.menu_pool.menu_pool``中注册它。

现在，当页面加载并生成菜单时，您的修饰符将能够检查和修改它的节点。

下面是一个简单的修饰符的例子，它将每个页面的``changed_by``属性放在相应的``NavigationNode``中::

    from menus.base import Modifier
    from menus.menu_pool import menu_pool

    from cms.models import Page

    class MyExampleModifier(Modifier):
        """
        This modifier makes the changed_by attribute of a page
        accessible for the menu system.
        """
        def modify(self, request, nodes, namespace, root_id, post_cut, breadcrumb):
            # only do something when the menu has already been cut
            if post_cut:
                # only consider nodes that refer to cms pages
                # and put them in a dict for efficient access
                page_nodes = {n.id: n for n in nodes if n.attr["is_page"]}
                # retrieve the attributes of interest from the relevant pages
                pages = Page.objects.filter(id__in=page_nodes.keys()).values('id', 'changed_by')
                # loop over all relevant pages
                for page in pages:
                    # take the node referring to the page
                    node = page_nodes[page['id']]
                    # put the changed_by attribute on the node
                    node.attr["changed_by"] = page['changed_by']
            return nodes

    menu_pool.register_modifier(MyExampleModifier)


它有一个方法:meth:`~menus.base.Modifier.modify`，
该方法应该返回:class:`~menus.base.NavigationNode`实例列表。
:meth:`~menus.base.Modifier.modify`应该接受以下参数:

``request``
  Django请求实例。要基于会话、用户或权限进行修改?

``nodes``
  所有的节点。通常，您希望再次返回它们。

``namespace``
  菜单名称空间。只有当某人请求一个只有来自这个名称空间的节点的菜单时才会给出。

``root_id``
  菜单请求是否基于ID?

``post_cut``
  每个修饰符调用两次。首先在整棵树上。在此之后，树被剪切为只显示当前菜单中显示的节点。
  切割之后，最后的树将再次调用修饰符。如果是这种情况，``post_cut``为 ``True``。

``breadcrumb``
  这是breadcrumb call而不是menu call吗?

下面是一个内置修改器的例子，它可以标记所有节点级别::


    class Level(Modifier):
        """
        marks all node levels
        """
        post_cut = True

        def modify(self, request, nodes, namespace, root_id, post_cut, breadcrumb):
            if breadcrumb:
                return nodes
            for node in nodes:
                if not node.parent:
                    if post_cut:
                        node.menu_level = 0
                    else:
                        node.level = 0
                    self.mark_levels(node, post_cut)
            return nodes

        def mark_levels(self, node, post_cut):
            for child in node.children:
                if post_cut:
                    child.menu_level = node.menu_level + 1
                else:
                    child.level = node.level + 1
                self.mark_levels(child, post_cut)

    menu_pool.register_modifier(Level)

Performance issues in menu modifiers
菜单修饰符中的性能问题
====================================

导航修饰符可能很快成为性能瓶颈。每个修饰符都被多次调用:对于面包屑(``breadcrumb=True``)，
对于整个菜单树(``post_cut=False``)，对于菜单树切到可见部分(``post_cut=True``)，或者对于导航的每个级别。
因此，在导航修饰符中执行低效的操作可能会导致较大的性能问题。保持修改器实现快速的一些技巧:

* 具体指定何时需要修饰符(在breadcrumb中，在剪切之前或之后)。
* 只考虑与修改相关的节点和页面。
* 执行尽可能少的数据库查询(即不在循环中执行)
* 在数据库查询中，获取您感兴趣的属性
* 如果您有多个修改要做，请尝试在相同的方法中应用它们。
