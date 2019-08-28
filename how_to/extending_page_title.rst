#################################
How to extend Page & Title models
如何扩展页面和标题模型
#################################

You can extend the :class:`cms.models.Page` and :class:`cms.models.Title` models with your own fields (e.g. adding an
icon for every page) by using the extension models: ``cms.extensions.PageExtension`` and
``cms.extensions.TitleExtension``, respectively.

通过分别使用扩展模型``cms.extensions.PageExtension`` and``cms.extensions.TitleExtension``:，
可以使用自己的字段扩展class:`cms.models.Page`和:class:`cms.models.Title`模型（例如，为每个页面添加图标）。


************************
Title vs Page extensions
************************

页面扩展名和标题扩展名之间的差异与``cms.models.page``和``cms.models.title``模型之间的差异有关。

* ``PageExtension``: 用于为页面的不同语言版本添加应该具有相同值的字段——例如，图标。
* ``TitleExtension``: 用于为页面的不同语言版本添加具有特定语言值的字段——例如，关键字。


***************************
Implement a basic extension
实现基本扩展
***************************

需要三个基本步骤：

* add the extension *model* 添加扩展模型
* add the extension *admin* 添加扩展管理
* add a toolbar menu item for the extension 为扩展添加工具栏菜单项


Page model extension example
页面模型扩展示例
============================

The model
---------

要向页面模型添加字段，请创建一个继承自``cms.extensions.PageExtension``的类。
您的类应该位于应用程序的一个``models.py``(或模块)中。


.. note::

    因为``PageExtension``(和``TitleExtension``)继承自``django.db.models.Model``，您可以随意添加任何您想要的字段，
    但请确保您没有对任何添加的字段使用惟一约束，因为惟一性会阻止扩展的复制机制正确工作。
    这意味着您不能在扩展模型上使用一对一关系。

最后，您需要使用``extension_pool``注册模型。

下面是一个简单的例子，它将``icon``字段添加到页面:

    from django.db import models
    from cms.extensions import PageExtension
    from cms.extensions.extension_pool import extension_pool


    class IconExtension(PageExtension):
        image = models.ImageField(upload_to='icons')


    extension_pool.register(IconExtension)

当然，您需要为这个新模型进行迁移并运行迁移。


The admin
---------

要使您的扩展可编辑，您必须首先创建一个admin类，该类继承``cms.extensions.PageExtensionAdmin``类。此管理员处理页面权限。

.. note::

    如果希望使用自己的admin类，请确保在queryset上使用 
    ``filter(extended_object__publisher_is_draft=True)`` 
    排除扩展的活动版本。

继续上面的示例模型，下面是一个简单的对应的``PageExtensionAdmin``类:

    from django.contrib import admin
    from cms.extensions import PageExtensionAdmin

    from .models import IconExtension


    class IconExtensionAdmin(PageExtensionAdmin):
        pass

    admin.site.register(IconExtension, IconExtensionAdmin)


由于PageExtensionAdmin继承自 ``ModelAdmin``, 所以您将能够使用适合您需要的Django ModelAdmin属性集。

.. note::

    注意，保存扩展名和CMS页面之间关系的字段是不可编辑的，因此它不会直接出现在页面管理视图中。
    这可能会在以后的更新中解决，但同时工具栏提供了对它的访问。


The toolbar item
工具栏项
----------------

您还需要使模型可从cms工具栏编辑，以便将扩展模型的每个实例与页面关联起来。

要为扩展添加工具栏项，请在其中一个应用程序中创建一个名为``cms_toolbars.py``的文件，并在每个页面上为扩展添加相关的菜单项。

下面是我们示例的一个简单版本。此示例将节点添加到现有页面菜单中，称为页面图标。
选中后，将打开一个模态对话框，其中可以编辑页面图标字段。

::

    from cms.toolbar_pool import toolbar_pool
    from cms.extensions.toolbar import ExtensionToolbar
    from django.utils.translation import ugettext_lazy as _
    from .models import IconExtension


    @toolbar_pool.register
    class IconExtensionToolbar(ExtensionToolbar):
        # defines the model for the current toolbar
        model = IconExtension

        def populate(self):
            # setup the extension toolbar with permissions and sanity checks
            current_page_menu = self._setup_extension_toolbar()

            # if it's all ok
            if current_page_menu:
                # retrieves the instance of the current extension (if any) and the toolbar item URL
                page_extension, url = self.get_page_extension_admin()
                if url:
                    # adds a toolbar item in position 0 (at the top of the menu)
                    current_page_menu.add_modal_item(_('Page Icon'), url=url,
                        disabled=not self.toolbar.edit_mode_active, position=0)


Title model extension example
标题模型扩展示例
=============================

在本例中，我们将创建一个评级扩展字段，它可以应用于每个标题，换句话说，应用于每个页面的每个语言版本。

..  note::

    请参考上面关于页面模型扩展示例的更详细的讨论，特别是特别说明。


The model
---------

::

    from django.db import models
    from cms.extensions import TitleExtension
    from cms.extensions.extension_pool import extension_pool


    class RatingExtension(TitleExtension):
        rating = models.IntegerField()


    extension_pool.register(RatingExtension)


The admin
---------

::

    from django.contrib import admin
    from cms.extensions import TitleExtensionAdmin
    from .models import RatingExtension


    class RatingExtensionAdmin(TitleExtensionAdmin):
        pass


    admin.site.register(RatingExtension, RatingExtensionAdmin)


The toolbar item
----------------

在本例中，我们需要循环页面的标题，并用这些标题填充菜单。

::

    from cms.toolbar_pool import toolbar_pool
    from cms.extensions.toolbar import ExtensionToolbar
    from django.utils.translation import ugettext_lazy as _
    from .models import RatingExtension
    from cms.utils import get_language_list  # needed to get the page's languages
    @toolbar_pool.register
    class RatingExtensionToolbar(ExtensionToolbar):
        # defines the model for the current toolbar
        model = RatingExtension

        def populate(self):
            # setup the extension toolbar with permissions and sanity checks
            current_page_menu = self._setup_extension_toolbar()

            # if it's all ok
            if current_page_menu and self.toolbar.edit_mode_active:
                # create a sub menu labelled "Ratings" at position 1 in the menu
                sub_menu = self._get_sub_menu(
                    current_page_menu, 'submenu_label', 'Ratings', position=1
                    )

                # retrieves the instances of the current title extension (if any)
                # and the toolbar item URL
                urls = self.get_title_extension_admin()

                # we now also need to get the titleset (i.e. different language titles)
                # for this page
                page = self._get_page()
                titleset = page.title_set.filter(language__in=get_language_list(page.node.site_id))

                # create a 3-tuple of (title_extension, url, title)
                nodes = [(title_extension, url, title.title) for (
                    (title_extension, url), title) in zip(urls, titleset)
                    ]

                # cycle through the list of nodes
                for title_extension, url, title in nodes:

                    # adds toolbar items
                    sub_menu.add_modal_item(
                        'Rate %s' % title, url=url, disabled=not self.toolbar.edit_mode_active
                        )



****************
Using extensions
使用扩展
****************

In templates
============

要在页模板中访问页扩展，只需访问适当的``related_name``字段，该字段现在在页对象中可用。


Page extensions
页面的扩展
---------------

As per the normal related_name naming mechanism, the appropriate field to
access is the same as your ``PageExtension`` model name, but lowercased. Assuming
your Page Extension model class is ``IconExtension``, the relationship to the
page extension model will be available on ``page.iconextension``. From there
you can access the extra fields you defined in your extension, so you can use
something like::
根据常规的related_name命名机制，要访问的适当字段与页面扩展模型名称相同，但是要小写。
假设您的页面扩展模型类是``IconExtension``，
那么与页面扩展模型的关系将在 ``page.iconextension``上可用。从那里，您可以访问您在扩展中定义的额外字段，
所以您可以使用以下内容:

    {% load staticfiles %}

    {# rest of template omitted ... #}

    {% if request.current_page.iconextension %}
        <img src="{% static request.current_page.iconextension.image.url %}">
    {% endif %}

``request.current_page`` 是访问呈现模板的当前页面的常规方法。

重要的是要记住，除非操作符已经为每个页面分配了一个页面扩展名，否则一个页面可能没有``iconextension``关系可用，
因此使用``{% if…%}…{% endif %}``以上。


Title extensions
标题扩展
----------------

为了在模板中检索标题扩展,使用``request.current_page.get_title_obj``获取标题对象。使用上面的例子，我们可以使用:

    {{ request.current_page.get_title_obj.ratingextension.rating }}


With menus
==========

与大多数其他页面属性一样，扩展名在菜单导航节点中没有表示，因此菜单模板在默认情况下无法访问它们。

为了使扩展可访问，您需要创建一个菜单修饰符(参见提供的示例)来实现这一点。:ref:`menu modifier
<integration_modifiers>` 。

每个页面扩展实例与其页面都有一对一的关系。通过使用反向关系，沿着``extension = page.yourextensionlowercased``行获取扩展。
然后将``page``的这个属性放在节点上——例如，作为``node.extension``。

在菜单模板中，我们在上面创建的图标扩展可以作为``child.extension.icon``使用。


Handling relations
处理关系
==================

如果``PageExtension``或``TitleExtension``包含来自另一个模型的外键或ManyToManyField，
还应该覆盖``copy_relations``方法(self、oldinstance、language)，
以便在CMS复制扩展以支持版本控制时适当地复制这些字段。


下面是一个使用``ManyToManyField``的例子 ::

    from django.db import models
    from cms.extensions import PageExtension
    from cms.extensions.extension_pool import extension_pool


    class MyPageExtension(PageExtension):

        page_categories = models.ManyToManyField(Category, blank=True)

        def copy_relations(self, oldinstance, language):
            for page_category in oldinstance.page_categories.all():
                page_category.pk = None
                page_category.mypageextension = self
                page_category.save()

    extension_pool.register(MyPageExtension)



********************
Complete toolbar API
********************

上面的示例使用了简化的工具栏API :ref:`simplified_extension_toolbar`.

.. _complete_toolbar_api:

如果你需要完全控制你的扩展工具栏项目的布局，你仍然可以使用底层API根据你的需要编辑工具栏:

    from cms.api import get_page_draft
    from cms.toolbar_pool import toolbar_pool
    from cms.toolbar_base import CMSToolbar
    from cms.utils import get_cms_setting
    from cms.utils.page_permissions import user_can_change_page
    from django.urls import reverse, NoReverseMatch
    from django.utils.translation import ugettext_lazy as _
    from .models import IconExtension


    @toolbar_pool.register
    class IconExtensionToolbar(CMSToolbar):
        def populate(self):
            # always use draft if we have a page
            self.page = get_page_draft(self.request.current_page)

            if not self.page:
                # Nothing to do
                return

            if user_can_change_page(user=self.request.user, page=self.page):
                try:
                    icon_extension = IconExtension.objects.get(extended_object_id=self.page.id)
                except IconExtension.DoesNotExist:
                    icon_extension = None
                try:
                    if icon_extension:
                        url = reverse('admin:myapp_iconextension_change', args=(icon_extension.pk,))
                    else:
                        url = reverse('admin:myapp_iconextension_add') + '?extended_object=%s' % self.page.pk
                except NoReverseMatch:
                    # not in urls
                    pass
                else:
                    not_edit_mode = not self.toolbar.edit_mode_active
                    current_page_menu = self.toolbar.get_or_create_menu('page')
                    current_page_menu.add_modal_item(_('Page Icon'), url=url, disabled=not_edit_mode)


现在，当操作员从工具栏中调用"Edit this page..."将会有一个附加的菜单项 ``Page Icon ...`` ,
(在本例中)，它可用于打开一个模态对话框，在该对话框中，操作符可以影响新``icon``字段。

注意，当保存扩展名时，相应的页面被标记为有未发布的更改。要查看新扩展值发布页面。


.. _simplified_extension_toolbar:

Simplified Toolbar API
简化API工具栏
======================

简化的工具栏API通过从``ExtensionToolbar``派生工具栏类来工作，该工具栏类提供以下API:

* ``ExtensionToolbar.get_page_extension_admin()``: 对于页面扩展，检索相关工具栏项的正确管理URL;
    返回工具栏项的扩展实例(如果不存在，则返回None)和管理URL
* ``ExtensionToolbar.get_title_extension_admin()``: 对于标题扩展，检索相关工具栏项的正确管理URL;
    返回扩展实例列表(如果不存在，则返回None)和当前页面每个标题的管理url
