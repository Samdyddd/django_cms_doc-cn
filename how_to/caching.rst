#####################
How to manage caching 管理缓存
#####################


******
Set-up
******

要设置缓存，请在django中配置缓存后端。

缓存的详细信息可以在这里找到: https://docs.djangoproject.com/en/dev/topics/cache/

在您的中间件设置中，请确保首先添加了``django.middleware.cache.UpdateCacheMiddleware`` ，其次是
``django.middleware.cache.FetchFromCacheMiddleware``::

    MIDDLEWARE_CLASSES=[
            'django.middleware.cache.UpdateCacheMiddleware',
            ...
            'cms.middleware.language.LanguageCookieMiddleware',
            'cms.middleware.user.CurrentUserMiddleware',
            'cms.middleware.page.CurrentPageMiddleware',
            'cms.middleware.toolbar.ToolbarMiddleware',
            'django.middleware.cache.FetchFromCacheMiddleware',
        ],


Plugins
=======

.. versionadded:: 3.0

通常所有插件都会被缓存。
如果你有一个基于当前用户或请求的其他动态属性的动态插件，请在插件类上设置``cache=False``属性:

    class MyPlugin(CMSPluginBase):
        name = _("MyPlugin")
        cache = False

.. warning::
    如果禁用插件缓存，请确保重新启动服务器，并在之后清除缓存。

Content Cache Duration
======================

Default: 60

This can be changed in :setting:`CMS_CACHE_DURATIONS`

Settings
========

缓存默认设置为true。请参考以下设置，以 enable/disable各种缓存行为:

- :setting:`CMS_PAGE_CACHE`
- :setting:`CMS_PLACEHOLDER_CACHE`
- :setting:`CMS_PLUGIN_CACHE`





