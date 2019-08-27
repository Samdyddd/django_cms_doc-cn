.. _complex_apphooks_how_to:

###########################################
How to manage complex apphook configuration
如何管理复杂的apphook配置
###########################################

在如何创建 :ref:`apphooks_how_to` 我们讨论了使用apphook的一些基本要点。
在本文档中，我们将介绍一些更复杂的实现可能性。


.. _multi_apphook:

***************************************
Attaching an application multiple times
多次附加应用程序
***************************************

Define a namespace at class-level
在类级定义名称空间
=================================

如果您想将应用程序多次附加到不同的页面，那么定义apphook的类必须具有``app_name``属性:

    class MyApphook(CMSApp):
        name = _("My Apphook")
        app_name = "myapp"

        def get_urls(self, page=None, language=None, **kwargs):
            return ["myapp.urls"]

The ``app_name`` does three key things:

* 它为反向url的视图和模板提供回退名称空间。
* 当应用apphook时，它在页面管理中公开应用程序实例名字段。
* 它设置了默认的apphook实例名(您将在应用程序实例名字段中看到)。

我们将用一个例子来解释这些。假设应用程序的视图或模板使用
``reverse('myapp:index')`` or ``{% url 'myapp:index' %}``.

在这种情况下，您应用的任何apphooks的名称空间都必须匹配myapp。如果没有，使用它们的页面将抛出``NoReverseMatch``错误。

您可以在应用程序实例名称字段中为apphook实例设置名称空间。但是，如果已经存在具有该值的实例，则需要将其设置为不同的值。
在本例中，只要``app_name = "myapp"``就可以了;即使系统没有找到与实例名称匹配的实例，它也会返回到硬连接到类中的实例。

换句话说，正确地设置``app_name``可以确保url反转能够工作，因为它正确地设置了回退名称空间。


Set a namespace at instance-level
在实例级设置名称空间
=================================

另一方面，如果找到匹配项，应用程序实例名将覆盖``app_name``。

这种安排允许您在需要灵活性的情况下使用多个应用程序实例和名称空间，同时确保在不需要的情况下使用一种简单的方法使其工作。

Django's :ref:`django:topics-http-reversing-url-namespaces`的反向命名空间url文档提供了更多关于如何工作的信息，但是简化版是:

1. First, 它将尝试为应用程序实例名找到匹配项.
2. 如果失败，它将尝试为 ``app_name``找到匹配项。


.. _apphook_configurations:

**********************
Apphook configurations
**********************

命名空间您的apphook还可以管理额外的数据库存储apphook配置，以实例为基础。


Basic concepts
基本概念
==============

为了捕获apphook的不同实例可以接受的配置，
需要创建Django模型——每个apphook实例都是该模型的一个实例，并通过Django管理员以通常的方式进行管理

一旦设置好，apphook配置可以应用于apphook实例，在apphook实例所属页面的高级设置中:

.. image:: /how_to/images/select_apphook_configuration.png
   :alt: selecting an apphook configuration application
   :width: 400
   :align: center

然后在应用程序的视图中加载该名称空间的配置，并将用于确定它的行为方式。

创建应用程序配置实际上创建了一个apphook实例名称空间。
一旦创建，配置的名称空间就不能更改——如果需要不同的名称空间，还需要创建新的配置。


********************************
An example apphook configuration
一个apphook配置示例
********************************

为了说明这一切是如何工作的，我们将创建一个新的FAQ应用程序，它提供一个简单的问题和答案列表，
以及一个apphook类和一个apphook配置模型，该模型允许它以多种配置存在于站点的多个位置。

我们假设您已经运行了一个django CMS项目。

Using helper applications
使用辅助应用程序
=========================

在本例中，我们将使用几个简单的helper应用程序，以简化我们的工作。


Aldryn Apphooks Config
----------------------

`Aldryn Apphooks Config <https://github.com/aldryn/aldryn-apphooks-config>`_ 是一个帮助应用程序，它使开发可配置的Apphooks变得更容易。
例如，它为您提供了一个AppHookConfig来子类化，以及其他有用的组件来节省您的时间。

在本例中，我们将按照我们的建议使用Aldryn Apphooks配置。但是，您不必在自己的项目中使用它;如果您愿意，可以手工构建所需的代码。

Use ``pip install aldryn-apphooks-config`` to install it.

Aldryn Apphooks配置依次安装`Django AppData <https://github.com/ella/django-appdata>`_，
这为应用程序扩展另一个应用程序提供了一种优雅的方式;我们也会利用这个。


Create the new FAQ application
创建新的FAQ应用程序
==============================

.. code-block:: shell

    python manage.py startapp faq


Create the FAQ ``Entry`` model
------------------------------

``models.py``:

.. code-block:: python

    from aldryn_apphooks_config.fields import AppHookConfigField
    from aldryn_apphooks_config.managers import AppHookConfigManager
    from django.db import models
    from faq.cms_appconfig import FaqConfig


    class Entry(models.Model):
        app_config = AppHookConfigField(FaqConfig)
        question = models.TextField(blank=True, default='')
        answer = models.TextField()

        objects = AppHookConfigManager()

        def __unicode__(self):
            return self.question

        class Meta:
            verbose_name_plural = 'entries'

The ``app_config`` field is a ``ForeignKey`` to an apphook configuration model;
app_config字段是apphook配置模型的外键;我们马上就会创建它。
此模型将保存特定的名称空间配置，并使将每个FAQ条目分配到名称空间成为可能。

有了自定义``AppHookConfigManager`` ，可以使用方便的快捷方式过滤queryset条目``Entry.objects.namespace('foobar')``.

Define the AppHookConfig subclass
---------------------------------

In a new file ``cms_appconfig.py`` in the FAQ application:

.. code-block:: python

    from aldryn_apphooks_config.models import AppHookConfig
    from aldryn_apphooks_config.utils import setup_config
    from app_data import AppDataForm
    from django.db import models
    from django import forms
    from django.utils.translation import ugettext_lazy as _


    class FaqConfig(AppHookConfig):
        paginate_by = models.PositiveIntegerField(
            _('Paginate size'),
            blank=False,
            default=5,
        )


    class FaqConfigForm(AppDataForm):
        title = forms.CharField()
    setup_config(FaqConfigForm, FaqConfig)

实现可以完全空，因为最小模式已经在Aldryn Apphooks配置提供的抽象父模型中定义。

这里我们在model上定义了一个额外的字段 ``paginate_by``。稍后我们将使用它来控制每个页面应该显示多少条目。

我们还设置了一个 ``FaqConfigForm``, 它使用AppDataForm向FaqConfig添加一个字段，而不需要实际接触它的模型。

title字段也可以只是一个模型字段，比如paginate_by。但是我们使用AppDataForm来演示这个功能


Define its admin properties
定义它的管理属性
---------------------------

In ``admin.py`` 我们需要定义所有要显示的字段:

.. code-block:: python

    from django.contrib import admin
    from .cms_appconfig import FaqConfig
    from .models import Entry
    from aldryn_apphooks_config.admin import ModelAppHookConfig, BaseAppHookConfig


    class EntryAdmin(ModelAppHookConfig, admin.ModelAdmin):
        list_display = (
            'question',
            'answer',
            'app_config',
        )
        list_filter = (
            'app_config',
        )
    admin.site.register(Entry, EntryAdmin)


    class FaqConfigAdmin(BaseAppHookConfig, admin.ModelAdmin):
        def get_config_fields(self):
            return (
                'paginate_by',
                'config.title',
            )
    admin.site.register(FaqConfig, FaqConfigAdmin)

``get_config_fields`` d定义应该显示的字段。使用AppData表单的任何字段都需要使用 ``config.``.


Define the apphook itself
-------------------------

现在让我们创建apphook，并将其设置为支持多个实例. In ``cms_apps.py``:

.. code-block:: python

    from aldryn_apphooks_config.app_base import CMSConfigApp
    from cms.apphook_pool import apphook_pool
    from django.utils.translation import ugettext_lazy as _
    from .cms_appconfig import FaqConfig


    @apphook_pool.register
    class FaqApp(CMSConfigApp):
        name = _("Faq App")
        urls = ["faq.urls"]
        app_name = "faq"
        app_config = FaqConfig


Define a list view for FAQ entries
为FAQ条目定义一个列表视图
----------------------------------

我们已经准备好了所有的基础设施。现在，我们将为FAQ条目添加一个列表视图，它只显示当前使用名称空间的条目. In ``views.py``:

.. code-block:: python

    from aldryn_apphooks_config.mixins import AppConfigMixin
    from django.views import generic
    from .models import Entry


    class IndexView(AppConfigMixin, generic.ListView):
        model = Entry
        template_name = 'faq/index.html'

        def get_queryset(self):
            qs = super(IndexView, self).get_queryset()
            return qs.namespace(self.namespace)

        def get_paginate_by(self, queryset):
            try:
                return self.config.paginate_by
            except AttributeError:
                return 10


``AppConfigMixin`` 为您节省了在视图中设置任何属性的工作——它会自动设置视图类实例:

* 中的当前名称空间``self.namespace``
* 名称空间配置(FaqConfig的实例) in ``self.config``
* 传递给 in the ``current_app parameter`` 传递给 ``Response`` class

在本例中，我们只筛选``get_queryset``中分配给当前名称空间的条目.
由于前面定义的模型管理器，``qs.namespace``相当于``qs.filter(app_config__namespace=self.namespace)``.

In ``get_paginate_by`` 我们使用appconfig模型中的值.


Define a template
^^^^^^^^^^^^^^^^^

In ``faq/templates/faq/index.html``:

.. code-block:: html+django

    {% extends 'base.html' %}

    {% block content %}
        <h1>{{ view.config.title }}</h1>
        <p>Namespace: {{ view.namespace }}</p>
        <dl>
            {% for entry in object_list %}
                <dt>{{ entry.question }}</dt>
                <dd>{{ entry.answer }}</dd>
            {% endfor %}
        </dl>

        {% if is_paginated %}
            <div class="pagination">
                <span class="step-links">
                    {% if page_obj.has_previous %}
                        <a href="?page={{ page_obj.previous_page_number }}">previous</a>
                    {% else %}
                        previous
                    {% endif %}

                    <span class="current">
                        Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}.
                    </span>

                    {% if page_obj.has_next %}
                        <a href="?page={{ page_obj.next_page_number }}">next</a>
                    {% else %}
                        next
                    {% endif %}
                </span>
            </div>
        {% endif %}
    {% endblock %}


URLconf
^^^^^^^

``urls.py``:

.. code-block:: python

    from django.conf.urls import url
    from . import views


    urlpatterns = [
        url(r'^$', views.IndexView.as_view(), name='index'),
    ]


Put it all together
===================

Finally, we add ``faq`` to ``INSTALLED_APPS``, 然后创建并运行迁移:

.. code-block:: shell

    python manage.py makemigrations faq
    python manage.py migrate faq

Now we should be all set.

使用faq apphook创建两个页面(不要忘记发布它们)，使用不同的名称空间和不同的配置。还创建一些分配给两个名称空间的条目。

您可以尝试不同的配置行为(在本例中，只有分页可用)，以及不同条目实例与特定apphook关联的方式。
