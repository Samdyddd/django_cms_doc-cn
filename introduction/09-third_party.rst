.. _third_party:

#####################################
Integrating a third-party application 
#####################################

我们已经编写了自己的django CMS插件和应用程序，
但是现在我们想用第三方应用程序Aldryn News & Blog扩展CMS。


******************
Basic installation
******************

首先，我们需要从PyPI将app安装到我们的虚拟环境中:
`PyPI <https://pypi.python.org>`_::

    pip install aldryn-newsblog


***************
Django settings
***************

``INSTALLED_APPS``
==================

在settings.py中为INSTALLED_APPS添加应用程序及其任何尚未存在的需求。
有些人已经到场;你可以检查它们，因为你需要避免重复:

.. code-block:: python

    # you will probably need to add:
    'aldryn_apphooks_config',
    'aldryn_boilerplates',
    'aldryn_categories',
    'aldryn_common',
    'aldryn_newsblog',
    'aldryn_people',
    'aldryn_reversion',
    'djangocms_text_ckeditor',
    'parler',
    'sortedm2m',
    'taggit',

    # and you will probably find the following already listed:
    'easy_thumbnails',
    'filer',
    'reversion',


``THUMBNAIL_PROCESSORS``
========================

其中一个依赖项是Django Filer。它提供了一个特殊的功能，允许更复杂的图像裁剪。为了实现这一点，它需要自己的缩略图处理器
(``filer.thumbnail_processors.scale_and_crop_with_subject_location``) to be listed in
``settings.py`` in place of ``easy_thumbnails.processors.scale_and_crop``:

.. code-block:: python
   :emphasize-lines: 4,5

    THUMBNAIL_PROCESSORS = (
        'easy_thumbnails.processors.colorspace',
        'easy_thumbnails.processors.autocrop',
        # 'easy_thumbnails.processors.scale_and_crop',  # disable this one
        'filer.thumbnail_processors.scale_and_crop_with_subject_location',
        'easy_thumbnails.processors.filters',
    )


``ALDRYN_BOILERPLATE_NAME``
===========================

Aldryn News & Blog使用Aldryn样板文件为不同的CSS框架提供多组模板和静态文件。
我们在本教程中使用Bootstrap 3，所以让我们通过添加设置来选择bootstrap3:

.. code-block:: python

    ALDRYN_BOILERPLATE_NAME='bootstrap3'


``STATICFILES_FINDERS``
=======================

将样板文件静态文件查找器添加到 ``STATICFILES_FINDERS``, *immediately before*
``django.contrib.staticfiles.finders.AppDirectoriesFinder``:

.. code-block:: python
   :emphasize-lines: 3

    STATICFILES_FINDERS = [
        'django.contrib.staticfiles.finders.FileSystemFinder',
        'aldryn_boilerplates.staticfile_finders.AppDirectoriesFinder',
        'django.contrib.staticfiles.finders.AppDirectoriesFinder',
    ]

如果在settings.py中没有定义STATICFILES_FINDERS，则复制并粘贴上面的代码。


``TEMPLATES``
=============

.. important::

    在Django 1.8中， ``TEMPLATE_LOADERS`` and ``TEMPLATE_CONTEXT_PROCESSORS`` 
    设置被卷进了模板设置中 ``TEMPLATES`` setting 。我们假设这里使用的是Django 1.8。


.. code-block:: python
   :emphasize-lines: 7,11

    TEMPLATES = [
        {
            # ...
            'OPTIONS': {
                'context_processors': [
                    # ...
                    'aldryn_boilerplates.context_processors.boilerplate',
                    ],
                'loaders': [
                    # ...
                    'aldryn_boilerplates.template_loaders.AppDirectoriesLoader',
                    ],
                },
            },
        ]


********************
Migrate the database
********************

我们添加了一个新的应用程序，所以我们需要更新我们的数据库::

    python manage.py migrate

重新启动服务器


***************************
Create a new apphooked page
***************************

News & Blog应用程序附带django CMS apphook，所以可以添加一个新的django CMS页面(称为News)，
并将News & Blog应用程序添加到其中，就像您在民意调查中所做的那样

对于这个应用程序，我们还需要创建并选择一个应用程序配置。

给这个应用程序配置一些设置:

* ``Instance namespace``: *news* (this is used for reversing URLs)
* ``Application title``: *News* (the name that will represent the application configuration in the
  admin)
* ``Permalink type``: choose a format you prefer for news article URLs

保存此应用程序配置，并确保在应用程序配置中选择了它。

发布新页面，您应该会发现News & Blog应用程序正在那里工作。
(在实际创建任何文章之前，它只会告诉您没有可用的项目。)

****************************
Add new News & Blog articles
****************************

您可以使用admin或new News菜单添加新文章，当您在属于News & Blog的页面上时，该菜单现在出现在工具栏中。

你也可以在另一个页面中插入一个最新的文章插件——就像所有优秀的django CMS应用程序一样，Aldryn News & Blog也带有插件。


.. _aldryn-boilerplates: https://github.com/aldryn/aldryn-boilerplates
