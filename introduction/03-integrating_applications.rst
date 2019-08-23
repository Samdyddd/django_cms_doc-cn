.. _integrating_applications:

########################
Integrating applications
########################

本教程的所有后续部分都关注将其他应用程序集成到django CMS中的不同方法。
其他应用程序可以轻松地构建到django CMS站点中，这是该系统的一个重要特性。

集成应用程序并不仅仅意味着将它们与django CMS一起安装，以便它们能够和平共存。
这意味着使用django CMS的特性将它们构建成一个统一的web项目，从而加快管理站点的工作，
并使更丰富和更自动化的发布成为可能。

django CMS集成工作的关键在于，它不需要修改其他应用程序，除非您愿意。
当您使用第三方应用程序，并且不想维护自己的分叉版本时，这一点尤其重要。
(唯一的例外是，如果您决定将django CMS特性直接构建到应用程序本身，
例如在其他应用程序中使用占位符时 :ref:`placeholders in other applications<placeholders_outside_cms>`.)

在本教程中，我们将使用一个基本的Django民意调查应用程序并将其集成到CMS中。

So we will:

* 将民意调查应用程序合并到项目中
* 创建第二个独立的Polls/CMS集成应用程序来管理集成

通过这种方式，我们可以集成民意调查应用程序，而无需更改其中的任何内容。


*************************************
Incorporate the ``polls`` application
*************************************

Install ``polls``
=================

使用pip从其GitHub存储库安装应用程序: ``pip``::

    pip install git+http://git@github.com/divio/django-polls.git#egg=polls

让我们将这个应用程序添加到我们的项目中。
在项目的settings.py中，在INSTALLED_APPS的末尾添加'polls'(请参阅INSTALLED_APPS设置订购的说明)

将``poll`` URL配置添加到项目的``urls.py``中的``urlpatterns``:

..  code-block:: python
    :emphasize-lines: 3

    urlpatterns += i18n_patterns(
        url(r'^admin/', include(admin.site.urls)),
        url(r'^polls/', include('polls.urls')),
        url(r'^', include('cms.urls')),
    )

注意，它必须包含在django CMS url的行之前。
django CMS的URL模式需要放在最后，因为它“吞噬”了任何之前的模式没有匹配的内容。

现在运行应用程序的迁移:

.. code-block:: bash

    python manage.py migrate polls

此时，您应该能够登录Django admin- ``http://localhost:8000/admin/`` -并找到poll应用程序。

.. image:: /introduction/images/polls-admin.png
   :alt: the polls application admin
   :width: 400
   :align: center

创建一个新的 **Poll**, for example:

* **Question**: *Which browser do you prefer?*

  **Choices**:

    * *Safari*
    * *Firefox*
    * *Chrome*

现在，如果访问 ``http://localhost:8000/en/polls/``, 应该能够看到已发布的民意调查并提交响应。
and submit a response.

.. image:: /introduction/images/polls-unintegrated.png
   :alt: the polls application
   :width: 400
   :align: center


改进投票模板
===============================

您会注意到，在poll应用程序中，我们只有很少的模板，没有导航或样式。

另一方面，django CMS页面可以访问项目中的许多默认模板，所有这些模板都扩展了一个名为base.html的模板。
因此，让我们通过覆盖poll应用程序的基本模板来改进这一点。

我们将在项目目录中执行此操作

In ``mysite/templates``, add ``polls/base.html``, containing:

.. code-block:: html+django

    {% extends 'base.html' %}

    {% block content %}
        {% block polls_content %}
        {% endblock %}
    {% endblock %}

再次刷新/polls/页面，现在应该正确地集成到站点中。

.. image:: /introduction/images/polls-integrated.png
   :alt: the polls application, integrated
   :width: 400
   :align: center



**************************************************
Set up a new ``polls_cms_integration`` application
**************************************************

然而，到目前为止，poll应用程序已经集成到项目中，但还没有集成到django CMS本身。
这两个应用程序是完全独立的。它们不能使用彼此的数据或功能。

让我们创建新的``Polls/CMS``集成应用程序，将它们放在一起。

Create the application
======================

在项目根目录下创建一个名为``polls_cms_integration``的新包::

    python manage.py startapp polls_cms_integration

我们的工作区现在看起来是这样的::

    tutorial-project/
        media/
        mysite/
        polls_cms_integration/  # the newly-created application
            __init__.py
            admin.py
            models.py
            migrations.py
            tests.py
            views.py
        static/
        manage.py
        project.db
        requirements.txt


Add it to ``INSTALLED_APPS``
============================

下一步是将``polls_cms_integration``应用程序集成到项目中。


将polls_cms_integration添加到settings.py中的INSTALLED_APPS中——现在我们准备使用它开始将poll与django CMS集成。
我们将从开发一个Polls插件开始。
.. note::

    **向项目或应用程序添加模板?**

    早些时候，我们向项目添加了新的模板。我们也可以在polls_cms_integration中添加模板/polls/base.html。毕竟，这是我们要做的所有其他的积分工作。

    但是，现在我们有了一个应用程序，它对应该扩展的模板的名称进行了假设(请参阅我们创建的base.html模板的第一行)，这可能不适用于其他项目。

    此外，我们还必须确保polls_cms_integration先于INSTALLED_APPS中的轮询，否则polls_cms_integration中的模板实际上不会覆盖轮询中的模板。
    将它们放在项目中可以保证它们将覆盖所有应用程序中的那些。

    任何一种方法都是合理的，只要你理解它们的含义。
