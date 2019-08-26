.. _custom-plugins:

#####################
How to create Plugins 如何创建插件
#####################

CMS插件是可重用的内容发布者，可以插入django CMS页面(或者任何使用django CMS占位符的内容)。
它们使信息自动发布成为可能，无需进一步干预。

这意味着您发布的web内容(无论它是什么)始终保持最新。

It's like magic, but quicker.

除非您足够幸运地发现您的需求可以通过内置插件或许多可用的第三方插件来满足，
否则您必须编写自己的定制CMS插件。不过不要担心——编写CMS插件非常简单。


*************************************
Why would you need to write a plugin?
*************************************

插件是将另一个Django应用程序的内容集成到Django CMS页面中最方便的方法。

例如，假设您正在用django CMS为一家唱片公司开发一个站点。您可能希望在站点的主页上有一个“最新版本”框。

当然，您可以经常编辑该页面并更新信息。不过，一家明智的唱片公司也会用Django来管理自己的音乐目录，
这意味着Django已经知道本周的新专辑是什么。

这是一个很好的机会,利用这些信息来让你的生活更容易,所有您需要做的是创建一个django CMS插件,
您可以插入你的主页,并把它出版的最新版本信息的工作。

插件是可重用的。也许你的唱片公司正在重新发行一系列具有开创性的瑞士朋克唱片;在您的站点关于该系列的页面上，
您可以插入相同的插件，只是配置略有不同，它将发布关于该系列最新版本的信息。

********
Overview 概述
********

django CMS插件基本上由三部分组成

* a plugin **editor**, 插件编辑器，用于在每次部署插件时配置插件
* a plugin **publisher**, 插件发布程序，自动决定发布什么
* a plugin **template**, 插件模板，用于将信息呈现到web页面中

These correspond to the familiar Model-View-Template scheme:
这些对应于我们熟悉的模型-视图-模板模式:

* the plugin **model** to store its configuration 存储配置的插件模型
* the plugin **view** that works out what needs to be displayed 插件视图，它计算出需要显示什么
* the plugin **template** to render the information 插件模板来渲染信息

要创建你的插件，你需要:

* 一个:class:`cms.models.pluginmodel.CMSPlugin`子类。CMSPlugin用于存储插件实例的配置
* 一个:class:`cms.plugin_base.CMSPluginBase`的子类。CMSPluginBase定义插件的操作逻辑
* 呈现插件的模板

A note about :class:`cms.plugin_base.CMSPluginBase`
===================================================

:class:`cms.plugin_base.CMSPluginBase` is actually a sub-class of
:class:`django:django.contrib.admin.ModelAdmin`.

由于:class:`~cms.plugin_base.CMSPluginBase`子类``ModelAdmin``, 
CMS插件开发人员也可以使用几个重要的ModelAdmin选项。这些选项经常使用:

* ``exclude``
* ``fields``
* ``fieldsets``
* ``form``
* ``formfield_overrides``
* ``inlines``
* ``radio_fields``
* ``raw_id_fields``
* ``readonly_fields``

但是请注意，并不是所有的``ModelAdmin``选项在CMS插件中都是有效的。
特别是，ModelAdmin的变更列表所专用的任何选项都没有效果。这些和其他值得注意的选项被CMS忽略:

* ``actions``
* ``actions_on_top``
* ``actions_on_bottom``
* ``actions_selection_counter``
* ``date_hierarchy``
* ``list_display``
* ``list_display_links``
* ``list_editable``
* ``list_filter``
* ``list_max_show_all``
* ``list_per_page``
* ``ordering``
* ``paginator``
* ``prepopulated_fields``
* ``preserve_fields``
* ``save_as``
* ``save_on_top``
* ``search_fields``
* ``show_full_result_count``
* ``view_on_site``


An aside on models and configuration
关于模型和配置的题外话
====================================

The plugin **model**, the sub-class of :class:`cms.models.pluginmodel.CMSPlugin`,
实际上是可选的。

您可以有一个不需要配置的插件，因为它只做一件事。

例如，您可以有一个插件，它只发布关于过去七天最畅销记录的信息。很明显，
这不是一个很灵活的方法——你不能在上个月最畅销的版本中使用相同的插件。

通常，您会发现能够配置插件是很有用的，这需要一个模型。


*******************
The simplest plugin
最简单的插件
*******************

您可以使用``python manage.py startapp``为插件应用程序设置基本布局(请记住将插件添加到``INSTALLED_APPS``中)。
或者，只需要向现有的Django应用程序添加一个名为``cms_plugins.py``的文件。

In ``cms_plugins.py``, 你把你的插件。在我们的例子中，包括以下代码::

    from cms.plugin_base import CMSPluginBase
    from cms.plugin_pool import plugin_pool
    from cms.models.pluginmodel import CMSPlugin
    from django.utils.translation import ugettext_lazy as _

    @plugin_pool.register_plugin
    class HelloPlugin(CMSPluginBase):
        model = CMSPlugin
        render_template = "hello_plugin.html"
        cache = False

差不多做完了。剩下的就是添加模板。将以下内容添加到名为``hello_plugin.html``的文件的根模板目录中:

.. code-block:: html+django

    <h1>Hello {% if request.user.is_authenticated %}{{ request.user.first_name }} {{ request.user.last_name}}{% else %}Guest{% endif %}</h1>

这个插件将会在你的网站上通过用户的名字来欢迎他们，如果他们已经登录，或者作为客人，如果他们没有登录。

现在让我们仔细看看我们做了什么。您应该在``cms_plugins.py``文件中定义:class:`cms.plugin_base.CMSPluginBase`的子类。这
些类定义了不同的插件。

这些类有两个必需的属性:

* ``model``: 您希望用于存储有关此插件的信息的模型。如果您不需要为您的插件存储任何特殊的信息，
    例如配置，您可以简单地使用cms.models.pluginmodel。CMSPlugin(我们稍后会更仔细地研究这个模型)。
    在普通的admin类中，您不需要提供此信息，因为admin.site。register(Model, Admin)负责它，但是插件不是以这种方式注册的。
* ``name``: 在管理中显示的插件名称。通常，使用:func:`django.utils.translation.ugettext_lazy`将该字符串标记为可翻译，
  这是一种很好的实践，但是这是可选的。默认情况下，名称是类名称的更好版本。

如果 ``render_plugin``属性为True(默认值)，则必须定义以下属性之一:

* ``render_template``: 用来呈现此插件的模板.

**or**

* ``get_render_template``: 返回用于呈现插件的模板路径的方法。

除了这些属性之外，您还可以覆盖render()方法，该方法确定用于呈现插件的模板上下文变量。
默认情况下，此方法只向上下文添加实例和占位符对象，但是插件可以覆盖此方法以包含所需的任何上下文。

还有许多其他方法可用来覆盖CMSPluginBase子类。更多细节请参见 :class:`~cms.plugin_base.CMSPluginBase`


***************
Troubleshooting
***************

由于插件模块是由django的importlib找到并加载的，所以您可能会遇到错误，
因为运行时的路径环境不同。如果您的cms_plugins没有加载或无法访问，请尝试以下操作:

    $ python manage.py shell
    >>> from importlib import import_module
    >>> m = import_module("myapp.cms_plugins")
    >>> m.some_test_function()

.. _storing configuration:

*********************
Storing configuration 存储配置
*********************

在许多情况下，您希望为插件实例存储配置。例如，如果您有一个显示最新博客文章的插件，
您可能希望能够选择显示的条目数量。另一个例子是一个图库插件，您想要为插件选择要显示的图片。

为此，您可以通过子类化:class:`cms.models.pluginmodel.CMSPlugin`来创建Django模型。
已安装应用程序的``models.py``中的CMSPlugin。

让我们从上面改进``HelloPlugin``，为未经身份验证的用户配置它的回退名称。

在 ``models.py`` 中添加如下::

    from cms.models.pluginmodel import CMSPlugin

    from django.db import models

    class Hello(CMSPlugin):
        guest_name = models.CharField(max_length=50, default='Guest')


如果您学习了Django教程，这对您来说应该不是什么新鲜事。
与普通模型的唯一区别是，您可以子类化:class:`cms.models.pluginmodel.CMSPlugin` 而不是
:class:`django.db.models.Model`.

现在我们需要修改插件定义来使用这个模型，所以我们的新``cms_plugins.py``看起来像这样:

    from cms.plugin_base import CMSPluginBase
    from cms.plugin_pool import plugin_pool
    from django.utils.translation import ugettext_lazy as _

    from .models import Hello

    @plugin_pool.register_plugin
    class HelloPlugin(CMSPluginBase):
        model = Hello
        name = _("Hello Plugin")
        render_template = "hello_plugin.html"
        cache = False

        def render(self, context, instance, placeholder):
            context = super(HelloPlugin, self).render(context, instance, placeholder)
            return context

我们更改了``model``属性，以指向新创建的``Hello``模型，并将模型实例传递给上下文。

作为最后一步，我们必须更新我们的模板来使用这个新的配置:

.. code-block:: html+django

    <h1>Hello {% if request.user.is_authenticated %}
      {{ request.user.first_name }} {{ request.user.last_name}}
    {% else %}
      {{ instance.guest_name }}
    {% endif %}</h1>

我们唯一改变的是使用模板变量 ``{{ instance.guest_name }}`` 而不是else子句中的硬编码guest字符串。

.. warning::

    由于Django对子类模型使用隐式的一对一关系，您不能将您的模型字段命名为与任何已安装的插件相同的低大小写模型名称。
    如果您使用所有核心插件，这包括:文件、google地图、链接、图片、snippetptr、预告片、twittersearch、
    twitterrecententries和视频。

    此外，建议您避免使用page作为模型字段，因为它被声明为cms.models.pluginmodel的属性。
    如果没有进一步的工作，CMSPlugin和您的插件将无法在管理中正常工作。

.. warning::

    如果您使用的是python2。并覆盖模型文件的``_unicode__``方法，确保将其结果返回为UTF8-string。
    否则，在前端编辑器显示<Empty> plugin实例时，保存插件实例可能会失败。要在Unicode中返回，
    可以使用return语句``return u'{0}'.format(self.guest_name)``.

.. _handling-relations:

Handling Relations
==================

每次发布带有自定义插件的页面时，都会复制该插件。因此，如果你的自定义插件有外键(到它，或从它)或多对多关系，
你负责复制那些相关的对象，如果需要，每当CMS复制插件-它不会自动为你做这件事。

每个插件模型都继承基类中的空
:meth:`cms.models.pluginmodel.CMSPlugin.copy_relations` 方法，并在复制插件时调用该方法。
所以，你可以根据需要调整自己的目的。

通常，您希望它复制相关对象。为此，您应该在插件模型上创建一个名为``copy_relationships``的方法，
该方法接收插件的旧实例作为参数。

但是，您可能会决定不复制相关对象—例如，您可能希望不复制它们。或者，
你甚至可能想要为它选择一些完全不同的关系，或者当它被复制时创建一个新的关系…这取决于你的插件和你想要它工作的方式。

如果你想复制相关的对象，你需要用两种稍微不同的方式来做，这取决于你的插件是否与其他需要复制的对象有关系:

For foreign key relations *from* other objects
用于从其他对象获取外键关系
----------------------------------------------

您的插件可能有带有外键的项，如果您将其设置为其管理中的内联项，则通常会出现这种情况。
所以你可能有两个模型，一个用于插件，一个用于那些项目:::

    class ArticlePluginModel(CMSPlugin):
        title = models.CharField(max_length=50)

    class AssociatedItem(models.Model):
        plugin = models.ForeignKey(
            ArticlePluginModel,
            related_name="associated_item"
        )

然后你需要插件模型上的``copy_relationships()``方法来循环关联的项并复制它们，给新插件的副本外键::

    class ArticlePluginModel(CMSPlugin):
        title = models.CharField(max_length=50)

        def copy_relations(self, oldinstance):
            # Before copying related objects from the old instance, the ones
            # on the current one need to be deleted. Otherwise, duplicates may
            # appear on the public version of the page
            self.associated_item.all().delete()

            for associated_item in oldinstance.associated_item.all():
                # instance.pk = None; instance.pk.save() is the slightly odd but
                # standard Django way of copying a saved model instance
                associated_item.pk = None
                associated_item.plugin = self
                associated_item.save()

For many-to-many or foreign key relations *to* other objects
用于与其他对象的多对多或外键关系
------------------------------------------------------------

让我们假设这些是插件的相关部分::

    class ArticlePluginModel(CMSPlugin):
        title = models.CharField(max_length=50)
        sections = models.ManyToManyField(Section)

现在，当插件被复制时，你想要确保这些部分保持不变，所以它变成::

    class ArticlePluginModel(CMSPlugin):
        title = models.CharField(max_length=50)
        sections = models.ManyToManyField(Section)

        def copy_relations(self, oldinstance):
            self.sections = oldinstance.sections.all()

如果您的插件具有这两种类型的关系字段，您当然可能需要使用上面描述的两种复制技术。

Relations *between* plugins
---------------------------

当关系从一个插件到另一个插件时，管理它们的复制要困难得多。

See the GitHub issue `copy_relations() does not work for relations between cmsplugins #4143
<https://github.com/divio/django-cms/issues/4143>`_ for more details.
有关更多细节，请参见GitHub问题copy_relations()对cmsplugins #4143 <https://github.com/divio/django-cms/issues/4143>`_ 之间的关系不起作用。
********
Advanced
********

Inline Admin
============

If you want to have the foreign key relation as a inline admin, you can create an
``admin.StackedInline`` class and put it in the Plugin to "inlines". Then you can use the inline
admin form for your foreign key references::
如果希望将外键关系作为内联管理，可以创建一个``admin.StackedInline``类，并把它放在插件的“内联”。
然后，您可以使用内联管理表单为您的外键引用:

    class ItemInlineAdmin(admin.StackedInline):
        model = AssociatedItem


    class ArticlePlugin(CMSPluginBase):
        model = ArticlePluginModel
        name = _("Article Plugin")
        render_template = "article/index.html"
        inlines = (ItemInlineAdmin,)

        def render(self, context, instance, placeholder):
            context = super(ArticlePlugin, self).render(context, instance, placeholder)
            items = instance.associated_item.all()
            context.update({
                'items': items,
            })
            return context

Plugin form 插件形式
===========

Since :class:`cms.plugin_base.CMSPluginBase` extends
:class:`django:django.contrib.admin.ModelAdmin`, 
您可以为插件定制表单，就像定制管理接口一样

插件编辑机制使用的模板是
``cms/templates/admin/cms/page/plugin/change_form.html``. 你可能需要改变这一点。

如果你想定制这个，最好的方法是:

* 创建自己的模板，扩展 ``cms/templates/admin/cms/page/plugin/change_form.html``
  以提供所需的功能;
* 为:class:`cms.plugin_base.CMSPluginBase` sub-class with a
  ``change_form_template`` 子类提供一个指向新模板的更改表单模板属性。

扩展 ``admin/cms/page/plugin/change_form.html`` 可以确保在插件中保持统一的外观和功能

您可能希望这样做的原因有很多。例如，您可能有一个需要引用模板变量的javascript片段，
您可能会将该片段放在 ``{% block extrahead %}``中，放在``{{block.super }}``之后，以继承父模板中的现有项。


.. _custom-plugins-handling-media:

Handling media
==============

如果插件依赖于某些媒体文件、JavaScript或样式表，可以使用django-sekizai从插件模板中包含它们。
CMS模板总是强制使用css和js sekizai名称空间，因此应该使用它们来包含相应的文件。
有关dango -sekizai的更多信息，请参阅dango -sekizai文档。
`django-sekizai documentation`_.

请注意，sekizai无法帮助您使用管理端插件模板——以下是针对插件输出模板的。

Sekizai style
-------------

为了充分利用dango -sekizai的力量，对如何使用它有一个一致的风格是有帮助的。
下面是一组应该遵守的约定(但不一定要遵守):

* 每个addtoblock一个位。
  每个addtoblock总是包含一个外部CSS或JS文件，或者每个addtoblock包含一个代码片段。这是需要的，
* 外部文件应该在一行中，addtoblock标记和HTML标记之间没有空格或换行。
* 当使用嵌入式javascript或CSS时，HTML标记应该在换行符上。

A **good** example:

.. code-block:: html+django

    {% load sekizai_tags %}

    {% addtoblock "js" %}<script type="text/javascript" src="{{ MEDIA_URL }}myplugin/js/myjsfile.js"></script>{% endaddtoblock %}
    {% addtoblock "js" %}<script type="text/javascript" src="{{ MEDIA_URL }}myplugin/js/myotherfile.js"></script>{% endaddtoblock %}
    {% addtoblock "css" %}<link rel="stylesheet" type="text/css" href="{{ MEDIA_URL }}myplugin/css/astylesheet.css">{% endaddtoblock %}
    {% addtoblock "js" %}
    <script type="text/javascript">
        $(document).ready(function(){
            doSomething();
        });
    </script>
    {% endaddtoblock %}

A **bad** example:

.. code-block:: html+django

    {% load sekizai_tags %}

    {% addtoblock "js" %}<script type="text/javascript" src="{{ MEDIA_URL }}myplugin/js/myjsfile.js"></script>
    <script type="text/javascript" src="{{ MEDIA_URL }}myplugin/js/myotherfile.js"></script>{% endaddtoblock %}
    {% addtoblock "css" %}
        <link rel="stylesheet" type="text/css" href="{{ MEDIA_URL }}myplugin/css/astylesheet.css"></script>
    {% endaddtoblock %}
    {% addtoblock "js" %}<script type="text/javascript">
        $(document).ready(function(){
            doSomething();
        });
    </script>{% endaddtoblock %}


.. _plugin-context-processors:


Plugin Context
==============

插件可以访问django模板上下文。您可以使用with标记覆盖变量。

Example::

    {% with 320 as width %}{% placeholder "content" %}{% endwith %}


Plugin Context Processors
=========================

插件上下文处理器是可调用的，它在呈现之前修改所有插件的上下文。
它们是使用``CMS_PLUGIN_CONTEXT_PROCESSORS``设置启用的。

插件上下文处理器有3个参数:

* ``instance``: 插件模型的实例
* ``placeholder``: 此插件的占位符实例出现在。
* ``context``: 正在使用的上下文，包括请求。

返回值应该是一个字典，其中包含要添加到上下文中的任何变量。

Example::

    def add_verbose_name(instance, placeholder, context):
        '''
        This plugin context processor adds the plugin model's verbose_name to context.
        '''
        return {'verbose_name': instance._meta.verbose_name}



Plugin Processors 插件的处理器
=================

插件处理器是可调用的，它在呈现后修改所有插件的输出。它们是使用:setting:`CMS_PLUGIN_PROCESSORS`设置启用的。

插件处理器有4个参数:

* ``instance``: 插件模型的实例
* ``placeholder``: 此插件的占位符实例出现在
* ``rendered_content``: 包含插件呈现内容的字符串
* ``original_context``: 用于呈现插件的模板的原始上下文。
  the plugin.

.. note:: 插件处理器也应用于嵌入到文本插件中的插件(以及任何允许嵌套插件的自定义插件)。
          根据处理器的操作，这可能会破坏输出。
          例如，如果您的处理器将输出包装在div标记中，您可能会在p标记中包含div标记，这是无效的
          如果``instance._render_meta``没有改变返回``rendered_content``，就可以避免这种情况。
          ``text_enabled``为真，这是呈现嵌入式插件时的情况。
Example
-------

假设你想把每个插件包在主占位符的一个彩色框中，但是编辑每个插件的模板太复杂了:

In your ``settings.py``::

    CMS_PLUGIN_PROCESSORS = (
        'yourapp.cms_plugin_processors.wrap_in_colored_box',
    )

In your ``yourapp.cms_plugin_processors.py``::

    def wrap_in_colored_box(instance, placeholder, rendered_content, original_context):
        '''
        This plugin processor wraps each plugin's output in a colored box if it is in the "main" placeholder.
        '''
        # Plugins not in the main placeholder should remain unchanged
        # Plugins embedded in Text should remain unchanged in order not to break output
        if placeholder.slot != 'main' or (instance._render_meta.text_enabled and instance.parent):
            return rendered_content
        else:
            from django.template import Context, Template
            # For simplicity's sake, construct the template from a string:
            t = Template('<div style="border: 10px {{ border_color }} solid; background: {{ background_color }};">{{ content|safe }}</div>')
            # Prepare that template's context:
            c = Context({
                'content': rendered_content,
                # Some plugin models might allow you to customise the colors,
                # for others, use default colors:
                'background_color': instance.background_color if hasattr(instance, 'background_color') else 'lightyellow',
                'border_color': instance.border_color if hasattr(instance, 'border_color') else 'lightblue',
            })
            # Finally, render the content through that template, and return the output
            return t.render(c)


.. _Django admin documentation: http://docs.djangoproject.com/en/dev/ref/contrib/admin/
.. _django-sekizai: https://github.com/ojii/django-sekizai
.. _django-sekizai documentation: https://django-sekizai.readthedocs.io


Nested Plugins 嵌套的插件
==============

您可以将CMS插件嵌套在它们自己中。实现这一功能需要做几件事:

``models.py``:

.. code-block:: python

    class ParentPlugin(CMSPlugin):
        # add your fields here

    class ChildPlugin(CMSPlugin):
        # add your fields here


``cms_plugins.py``:

.. code-block:: python

    from .models import ParentPlugin, ChildPlugin

    @plugin_pool.register_plugin
    class ParentCMSPlugin(CMSPluginBase):
        render_template = 'parent.html'
        name = 'Parent'
        model = ParentPlugin
        allow_children = True  # This enables the parent plugin to accept child plugins
        # You can also specify a list of plugins that are accepted as children,
        # or leave it away completely to accept all
        # child_classes = ['ChildCMSPlugin']

        def render(self, context, instance, placeholder):
            context = super(ParentCMSPlugin, self).render(context, instance, placeholder)
            return context


    @plugin_pool.register_plugin
    class ChildCMSPlugin(CMSPluginBase):
        render_template = 'child.html'
        name = 'Child'
        model = ChildPlugin
        require_parent = True  # Is it required that this plugin is a child of another plugin?
        # You can also specify a list of plugins that are accepted as parents,
        # or leave it away completely to accept all
        # parent_classes = ['ParentCMSPlugin']

        def render(self, context, instance, placeholder):
            context = super(ChildCMSPlugin, self).render(context, instance, placeholder)
            return context


``parent.html``:

.. code-block:: html+django

    {% load cms_tags %}

    <div class="plugin parent">
        {% for plugin in instance.child_plugin_instances %}
            {% render_plugin plugin %}
        {% endfor %}
    </div>


`child.html`:

.. code-block:: html+django

    <div class="plugin child">
        {{ instance }}
    </div>


.. _extending_context_menus:

Extending context menus of placeholders or plugins
扩展占位符或插件的上下文菜单
==================================================

扩展占位符或插件的上下文菜单有三种可能性。

* 您可以扩展占位符上下文菜单。
* 您可以扩展所有插件上下文菜单。
* 您可以扩展当前插件上下文菜单。

为此，您可以在CMSPluginBase上覆盖3个方法。

* :meth:`~cms.plugin_base.CMSPluginBase.get_extra_placeholder_menu_items`
* :meth:`~cms.plugin_base.CMSPluginBase.get_extra_global_plugin_menu_items`
* :meth:`~cms.plugin_base.CMSPluginBase.get_extra_local_plugin_menu_items`

Example::

    class AliasPlugin(CMSPluginBase):
        name = _("Alias")
        allow_children = False
        model = AliasPluginModel
        render_template = "cms/plugins/alias.html"

        def render(self, context, instance, placeholder):
            context = super(AliasPlugin, self).render(context, instance, placeholder)
            if instance.plugin_id:
                plugins = instance.plugin.get_descendants(include_self=True).order_by('placeholder', 'tree_id', 'level',
                                                                                      'position')
                plugins = downcast_plugins(plugins)
                plugins[0].parent_id = None
                plugins = build_plugin_tree(plugins)
                context['plugins'] = plugins
            if instance.alias_placeholder_id:
                content = render_placeholder(instance.alias_placeholder, context)
                print content
                context['content'] = mark_safe(content)
            return context

        def get_extra_global_plugin_menu_items(self, request, plugin):
            return [
                PluginMenuItem(
                    _("Create Alias"),
                    reverse("admin:cms_create_alias"),
                    data={'plugin_id': plugin.pk, 'csrfmiddlewaretoken': get_token(request)},
                )
            ]

        def get_extra_placeholder_menu_items(self, request, placeholder):
            return [
                PluginMenuItem(
                    _("Create Alias"),
                    reverse("admin:cms_create_alias"),
                    data={'placeholder_id': placeholder.pk, 'csrfmiddlewaretoken': get_token(request)},
                )
            ]

        def get_plugin_urls(self):
            urlpatterns = [
                url(r'^create_alias/$', self.create_alias, name='cms_create_alias'),
            ]
            return urlpatterns

        def create_alias(self, request):
            if not request.user.is_staff:
                return HttpResponseForbidden("not enough privileges")
            if not 'plugin_id' in request.POST and not 'placeholder_id' in request.POST:
                return HttpResponseBadRequest("plugin_id or placeholder_id POST parameter missing.")
            plugin = None
            placeholder = None
            if 'plugin_id' in request.POST:
                pk = request.POST['plugin_id']
                try:
                    plugin = CMSPlugin.objects.get(pk=pk)
                except CMSPlugin.DoesNotExist:
                    return HttpResponseBadRequest("plugin with id %s not found." % pk)
            if 'placeholder_id' in request.POST:
                pk = request.POST['placeholder_id']
                try:
                    placeholder = Placeholder.objects.get(pk=pk)
                except Placeholder.DoesNotExist:
                    return HttpResponseBadRequest("placeholder with id %s not found." % pk)
                if not placeholder.has_change_permission(request):
                    return HttpResponseBadRequest("You do not have enough permission to alias this placeholder.")
            clipboard = request.toolbar.clipboard
            clipboard.cmsplugin_set.all().delete()
            language = request.LANGUAGE_CODE
            if plugin:
                language = plugin.language
            alias = AliasPluginModel(language=language, placeholder=clipboard, plugin_type="AliasPlugin")
            if plugin:
                alias.plugin = plugin
            if placeholder:
                alias.alias_placeholder = placeholder
            alias.save()
            return HttpResponse("ok")


.. _plugin-datamigrations-3.1:

Plugin data migrations 插件数据迁移
======================

由于3.1版中从Django MPTT迁移到Django -treebeard，这两个版本的插件模型有所不同。
模式迁移不会受到影响，因为迁移系统(包括South和Django)检测到不同的基础。

不过，数据迁移则是另一回事。

如果你的数据迁移做了如下事情:

.. code-block:: django

    MyPlugin = apps.get_model('my_app', 'MyPlugin')

    for plugin in MyPlugin.objects.all():
        ... do something ...

您可能会得到像
``django.db.utils.OperationalError: (1054, "Unknown column 'cms_cmsplugin.level' in 'field list'")``
因为根据迁移执行的顺序，历史模型可能与应用的数据库模式不同步

保持与3.0和3的兼容性。您可以强制数据迁移在django CMS迁移之前运行，django CMS迁移创建treebeard字段，
通过这样做，数据迁移将始终在“旧的”数据库模式上执行，并且不存在冲突。

For South migrations add this:

.. code-block:: django

    from distutils.version import LooseVersion
    import cms
    USES_TREEBEARD = LooseVersion(cms.__version__) >= LooseVersion('3.1')

    class Migration(DataMigration):

        if USES_TREEBEARD:
            needed_by = [
                ('cms', '0070_auto__add_field_cmsplugin_path__add_field_cmsplugin_depth__add_field_c')
            ]


对于Django迁移，请添加以下内容:

.. code-block:: django

    from distutils.version import LooseVersion
    import cms
    USES_TREEBEARD = LooseVersion(cms.__version__) >= LooseVersion('3.1')

    class Migration(migrations.Migration):

        if USES_TREEBEARD:
            run_before = [
                ('cms', '0004_auto_20140924_1038')
            ]
