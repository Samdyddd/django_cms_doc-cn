######################
How to create sitemaps 如何创建站点地图
######################


*******
Sitemap
*******

站点地图是谷歌使用的XML文件，通过使用它们的网站管理员工具并告诉它们站点地图的位置来索引您的网站。

The :class:`cms.sitemaps.CMSSitemap` 将创建一个包含CMS所有已发布页面的站点地图。


*************
Configuration
*************

 * add :mod:`django.contrib.sitemaps` to your project's :setting:`django:INSTALLED_APPS`
   setting
 * add ``from cms.sitemaps import CMSSitemap`` to the top of your main ``urls.py``
 * add ``from django.contrib.sitemaps.views import sitemap`` to ``urls.py```
 * add ``url(r'^sitemap\.xml$', sitemap, {'sitemaps': {'cmspages': CMSSitemap}}),``
   to your ``urlpatterns``


***************************
``django.contrib.sitemaps``
***************************

More information about :mod:`django.contrib.sitemaps` can be found in the official
`Django documentation <http://docs.djangoproject.com/en/dev/ref/contrib/sitemaps/>`_.


