.. _frontend-editable-fields:

#########################################################
How to enable frontend editing for Page and Django models
如何为页面和Django模型启用前端编辑
#########################################################

.. versionadded:: 3.0

除了占位符字段``PlaceholderFields``外，还可以通过Django CMS的前端编辑界面编辑“普通”Django模型字段(包括CMS页面和您自己的Django模型)。
这对于用户来说非常方便，因为它节省了在前端和管理视图之间切换的时间。

使用此接口，可以编辑的模型实例值在悬停时显示“双击编辑 Double-click to edit”提示。
双击会打开一个弹出窗口，其中包含该模型的更改表单。

.. note::

    这个界面目前还不能用于触摸屏用户，但是在将来的版本中会得到改进。

.. warning::

    此功能只与dango -hvad部分兼容: 使用
    ``render_model`` with hvad-translated fields (say
    ``{% render_model object 'translated_field' %}`` 如果hvad-enabled 对象不存在当前语言中会返回一个错误。
    作为一个工作区 ``render_model_icon`` 能被替代.

.. _render_model_templatetags:

*************
Template tags
*************

该特性依赖于共享公共代码的五个模板标记。所有这些都要求您在模板中加载``{% load
cms_tags %}``:

* :ttag:`render_model` (用于编辑特定字段)
* :ttag:`render_model_block` (用于编辑已定义块中的任何字段)
* :ttag:`render_model_icon` (用于编辑由另一个值表示的字段，如图像)
* :ttag:`render_model_add` (用于添加指定模型的实例)
* :ttag:`render_model_add_block` (用于添加指定模型的实例)

查看特定于标记的页面，以获得关于限制和警告的更详细的参考和讨论。

****************
Page titles edit
****************

对于CMS页面，您可以从前端编辑标题;根据指定的属性，也可以重写的默认字段将是可编辑的。

Main title::

    {% render_model request.current_page "title" %}


Page title::

    {% render_model request.current_page "page_title" %}

Menu title::

    {% render_model request.current_page "menu_title" %}

All three titles::

    {% render_model request.current_page "titles" %}


您总是可以通过提供``edit_field``参数自定义可编辑字段:::

    {% render_model request.current_page "title" "page_title,menu_title" %}


**************
Page menu edit 页面菜单编辑
**************

通过使用特殊的关键字changelist作为编辑字段，前端编辑将显示页面树;一种常见的模式是通过包装菜单模板标签来启用菜单中的更改:

.. code-block:: html+django

    {% render_model_block request.current_page "changelist" %}
        <h3>Menu</h3>
        <ul>
            {% show_menu 1 100 0 1 "sidebar_submenu_root.html" %}
        </ul>
    {% endrender_model_block %}

将渲染为:

.. code-block:: html+django

    <template class="cms-plugin cms-plugin-start cms-plugin-cms-page-changelist-1"></tempate>
        <h3>Menu</h3>
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/another">another</a></li>
            [...]
    <template class="cms-plugin cms-plugin-end cms-plugin-cms-page-changelist-1"></tempate>

.. warning:

    Be aware that depending on the layout of your menu templates, clickable
    area of the menu may completely overlap with the active area of the
    frontend editor thus preventing editing. In this case you may use
    ``{% render_model_icon %}``.
    The same conflict exists when menu template is managed by a plugin.

********************************
Editing 'ordinary' Django models
编辑“普通”的Django模型
********************************

如上所述，您自己的Django模型也可以在前端显示用于编辑的字段。
这是通过使用``FrontendEditableAdminMixin``基类实现的。

注意，这只对``PlaceholderFields``以外的字段是必需的。``PlaceholderFields``自动可用来进行前端编辑。

Configure the model's admin class
配置模型的admin类
=================================

通过添加``FrontendEditableAdminMixin`` 来配置您的管理类
(有关:mod:`Django的一般管理<django.contrib.admin>`信息，请参阅Django管理文档):

    from cms.admin.placeholderadmin import FrontendEditableAdminMixin
    from django.contrib import admin


    class MyModelAdmin(FrontendEditableAdminMixin, admin.ModelAdmin):
        ...

顺序很重要: 通常，mixins 必须放在第一位。

然后设置模板，在你想要公开模型进行编辑的地方，添加一个render_model模板标签:

    {% load cms_tags %}

    {% block content %}
    <h1>{% render_model instance "some_attribute" %}</h1>
    {% endblock content %}

See :ttag:`template tag reference <render_model>` for arguments documentation.
有关参数文档，请参阅模板标记引用。

Selected fields edit 选定字段编辑
====================

前端编辑也可以用于一组字段。

Set up the admin 设置管理员
----------------

你需要添加到你的模型管理元组字段可编辑从前端管理:

    from cms.admin.placeholderadmin import FrontendEditableAdminMixin
    from django.contrib import admin


    class MyModelAdmin(FrontendEditableAdminMixin, admin.ModelAdmin):
        frontend_editable_fields = ("foo", "bar")
        ...

Set up the template
-------------------

然后添加逗号分隔的字段列表(或仅一个字段的名称)到模板标签:

    {% load cms_tags %}

    {% block content %}
    <h1>{% render_model instance "some_attribute" "some_field,other_field" %}</h1>
    {% endblock content %}



Special attributes
==================

模板标签的属性参数不需要是模型字段，
属性或方法也可以用作目标;对于方法，将使用request作为参数调用它。


.. _custom-views:

Custom views 自定义视图
============

您可以将任何字段链接到自定义视图(不一定是管理视图)，以处理特定于模型的编辑工作流。

自定义视图可以作为已命名url (``view_url``参数)传递，也可以作为正在编辑的实例上的方法(或属性)的名称传递(``view_method``参数)。
在提供``view_method``的情况下，只要将``request``作为参数计算模板标记，就会调用它。

自定义视图不需要遵循任何特定的接口;它将获得``edit_fields``值作为一个``get``参数。

See :ttag:`template tag reference <render_model>` for arguments documentation.
有关参数文档，请参阅模板标记引用。

Example ``view_url``::

    {% load cms_tags %}

    {% block content %}
    <h1>{% render_model instance "some_attribute" "some_field,other_field" "" "admin:exampleapp_example1_some_view" %}</h1>
    {% endblock content %}


Example ``view_method``::

    class MyModel(models.Model):
        char = models.CharField(max_length=10)

        def some_method(self, request):
            return "/some/url"


    {% load cms_tags %}

    {% block content %}
    <h1>{% render_model instance "some_attribute" "some_field,other_field" "" "" "some_method" %}</h1>
    {% endblock content %}


Model changelist
================

通过使用特殊的关键字``changelist``作为编辑字段，前端编辑将显示模型变更列表:

.. code-block:: html+django

    {% render_model instance "name" "changelist" %}

Will render to:

.. code-block:: html+django

    <div class="cms-plugin cms-plugin-myapp-mymodel-changelist-1">
        My Model Instance Name
    </div>


.. filters:

*******
Filters
*******

如果需要对模板标签的输出值应用过滤器，请添加被引用的过滤器序列，如:ttag:`django:filter`模板标签中所示:

.. code-block:: html+django

    {% load cms_tags %}

    {% block content %}
    <h1>{% render_model instance "attribute" "" "" "truncatechars:9" %}</h1>
    {% endblock content %}



****************
Context variable
****************

模板标签输出可以保存在一个上下文变量中供以后使用，使用标准语法:

.. code-block:: html+django

    {% load cms_tags %}

    {% block content %}
    {% render_model instance "attribute" as variable %}

    <h1>{{ variable }}</h1>

    {% endblock content %}

