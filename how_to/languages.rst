.. _multilingual_support_how_to:

###############################
How to serve multiple languages  如何服务多种语言
###############################

如果您使用django CMS安装程序<https://github.com/nephila/djangocms-installer>`_启动您的项目，您会发现它已经被设置为提供多语言内容。
我们的如何安装django CMS手动指南也做了同样的事情。

本指南特别描述了启用多语言支持所需的步骤，以防您需要手动操作。

.. _multilingual_urls:

*****************
Multilingual URLs 多语种urls
*****************

如果使用不止一种语言，则需要通过:func:`~django.conf.urls.i18n.i18n_patterns`引用django CMS url，包括管理url。
有关此主题的更多信息，请参阅官方Django文档。<https://docs.djangoproject.com/en/dev/topics/i18n/translation/#internationalization-in-url-patterns>`_


完整示例 ``urls.py``::

    from django.conf import settings
    from django.conf.urls import include, url
    from django.contrib import admin
    from django.conf.urls.i18n import i18n_patterns, JavascriptCatalog
    from django.contrib.staticfiles.urls import staticfiles_urlpatterns

    admin.autodiscover()

    urlpatterns = [
        url(r'^jsi18n/(?P<packages>\S+?)/$', JavascriptCatalog.as_view()),
    ]

    urlpatterns += staticfiles_urlpatterns()

    # note the django CMS URLs included via i18n_patterns
    urlpatterns += i18n_patterns('',
        url(r'^admin/', admin.site.urls),
        url(r'^', include('cms.urls')),
    )


Monolingual URLs 单语的urls
================

当然，如果只需要单语url，不需要语言代码，那么就不要使用:func:`~django.conf.urls.i18n.i18n_patterns`:

    urlpatterns += [
        url(r'^admin/', admin.site.urls),
        url(r'^', include('cms.urls')),
    ]


************************************
Store the user's language preference 存储用户的语言首选项
************************************

用户的首选语言通过浏览会话来维护。因此django CMS必须存储在cookie中，以便在后续会话中记住用户的首选项。为了支持它,
必须将``cms.middleware.language.LanguageCookieMiddleware``添加到项目的``MIDDLEWARE_CLASSES``设置中

有关如何工作的更多信息，请参见django CMS如何确定使用哪种语言 :ref:`determining_language_preference`。

*********************
Working in templates
*********************

在页面中显示语言选择器
======================================

:ttag:`language_chooser` 将显示当前页面的语言选择器。 
您可以在 ``menu/language_chooser.html`` 中修改模板，
或者在必要时提供自己的模板。

Example:

.. code-block:: html+django

    {% load menu_tags %}
    {% language_chooser "myapp/language_chooser.html" %}


如果您在apphook中，并且拥有对象的详细视图，则可以将对象设置为视图中的工具栏。
cms将调用``get_absolute_url``在相应的语言为语言选择器:

Example:

.. code-block:: html+django

    class AnswerView(DetailView):
        def get(self, *args, **kwargs):
            self.object = self.get_object()
            if hasattr(self.request, 'toolbar'):
                self.request.toolbar.set_object(self.object)
            response = super(AnswerView, self).get(*args, **kwargs)
            return response


有了它，您可以更容易地控制将在语言选择器上返回什么url。

.. note::

    如果您有一个多语言对象，如果在``get_absolute_url``中没有该语言的翻译，请确保返回正确的url


Get the URL of the current page for a different language
获取用于不同语言的当前页面的URL
========================================================

The ``page_language_url`` 将当前页面的URL转换为另一种语言

Example:

.. code-block:: html+django

    {% page_language_url "de" %}


***************************************
Configuring language-handling behaviour
配置language-handling行为
***************************************

:setting:`CMS_LANGUAGES`描述了用于确定django CMS如何跨多种语言提供内容的所有选项。


.. _documentation: https://docs.djangoproject.com/en/dev/topics/i18n/translation/#internationalization-in-url-patterns
