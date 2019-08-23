.. _plugins_tutorial:

#######
Plugins
#######

在本教程中，我们将使用一个基本的Django民意调查应用程序并将其集成到CMS中。


*********************************
Create a plugin model
*********************************

In the ``models.py`` of ``polls_cms_integration`` add the following:

..  code-block:: python

    from django.db import models
    from cms.models import CMSPlugin
    from polls.models import Poll


    class PollPluginModel(CMSPlugin):
        poll = models.ForeignKey(Poll)

        def __str__(self):
            return self.poll.question


这创建了一个插件模型类;这些都继承自``cms.models.pluginmodel.CMSPlugin``基类。

.. note::

    django CMS plugins 继承了 :class:`cms.models.pluginmodel.CMSPlugin` (or a
    sub-class thereof) 而不是 :class:`models.Model <django.db.models.Model>`.

创建和运行迁移:

..  code-block:: bash

    python manage.py makemigrations polls_cms_integration
    python manage.py migrate polls_cms_integration


The Plugin Class
================

现在在``model .py``所在的文件夹中创建一个新文件``cms_plugin .py``。
plugin类负责为django CMS提供呈现插件所需的信息。

对于我们的poll插件，我们将编写以下插件类:

.. code-block:: python

    from cms.plugin_base import CMSPluginBase
    from cms.plugin_pool import plugin_pool
    from polls_cms_integration.models import PollPluginModel
    from django.utils.translation import ugettext as _


    @plugin_pool.register_plugin  # register the plugin
    class PollPluginPublisher(CMSPluginBase):
        model = PollPluginModel  # model where plugin data are saved
        module = _("Polls")
        name = _("Poll Plugin")  # name of the plugin in the interface
        render_template = "polls_cms_integration/poll_plugin.html"

        def render(self, context, instance, placeholder):
            context.update({'instance': instance})
            return context

.. note::

    所有插件类都必须继承自cms.plugin_base。CMSPluginBase必须在plugin_pool中注册它们自己。

一个合理的插件命名约定是:

* ``PollPluginModel``: the *model* class
* ``PollPluginPublisher``: the *plugin* class

你不需要遵循这个惯例，但要选择一个有意义的并坚持下去。


The template
============

插件类中的render_template属性是必需的，它告诉插件渲染时使用哪个render_template。

在这种情况下，模板需要位于 ``polls_cms_integration/templates/polls_cms_integration/poll_plugin.html``，
和如下所示:

.. code-block:: html+django

    <h1>{{ instance.poll.question }}</h1>

    <form action="{% url 'polls:vote' instance.poll.id %}" method="post">
        {% csrf_token %}
        <div class="form-group">
            {% for choice in instance.poll.choice_set.all %}
                <div class="radio">
                    <label>
                        <input type="radio" name="choice" value="{{ choice.id }}">
                        {{ choice.choice_text }}
                    </label>
                </div>
            {% endfor %}
        </div>
        <input type="submit" value="Vote" />
    </form>


***************************************************
Test the plugin
***************************************************

现在可以重新启动runserver，这是必需的，因为您添加了新的 ``cms_plugins.py`` 文件, and
visit http://localhost:8000/.

您现在可以将Poll插件放入任何页面的任何占位符中，就像您可以将其放入任何其他插件一样。

.. image:: /introduction/images/poll-plugin-in-menu.png
   :alt: the 'Poll plugin' in the plugin selector
   :width: 400
   :align: center

接下来，我们将把poll应用程序更全面地集成到django CMS项目中。
