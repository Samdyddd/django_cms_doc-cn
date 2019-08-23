.. _install-django-cms-tutorial:

#####################
Installing django CMS
#####################

我们将从设置环境开始。

************
要求
************

django CMS需要django 1.11或更新版本，Python 2.7或3.3或更新版本。本教程假设您使用的是python3。

************************
工作环境
************************

我们假设您安装了一个相当新的virtualenv版本，并且对它有一些基本的了解。


创建并激活virtual 环境
=========================================

::

    python3.6 -m venv env   # Python 2 usage: virtualenv env
    source env/bin/activate

注意，如果您正在使用Windows，请激活所需的virtualenv::

    env\Scripts\activate


在 virtual 环境中更新pip
=========================================

``pip``是Python安装程序。确保你的是最新的，因为早期的版本可能不太可靠:::

	pip install --upgrade pip


使用django CMS安装程序
============================

..  note::

    django CMS安装程序还不能用于django CMS 3.6或django 2或更高版本。
    本节将在django CMS 3.6的最终版本发布之前更新或删除。

django CMS安装程序是一个帮助设置新项目的脚本。
安装::

    pip install djangocms-installer

这为您提供了一个新命令, ``djangocms``.

创建一个新的目录，并将cd放入其中::

    mkdir tutorial-project
    cd tutorial-project

创建一个叫 ``mysite``的新Django项目::

    djangocms -f -p . mysite

This means:

* 运行django CMS安装程序
* install Django Filer too (``-f``) - **required for this tutorial**
* 使用当前目录作为新项目目录的父目录 (``-p .``)
* 调用新项目目录 ``mysite``

.. note:: **About Django Filer**

   Django Filer，一个用于管理文件和处理图像的有用应用程序。
   虽然django CMS本身并不需要它，但是大量django CMS插件都使用它，几乎所有django CMS项目都安装了它。
   如果你知道你不需要它，就省略掉它。
   有关更多信息，请参阅django CMS安装程序文档。 <https://djangocms-installer.readthedocs.io>`_.


.. warning::
   djangocms-installer期望目录。在这个阶段是空的，并会检查这个，如果不是，就会警告。
   你可以让它跳过检查，然后使用-s标志继续; 
   **注意，这可能会覆盖现有文件**.


Windows用户可能需要做一些额外的工作，以确保Python文件正确关联，如果这不能立即工作
    assoc .py=Python.file
    ftype Python.File="C:\Users\Username\workspace\demo\env\Scripts\python.exe" "%1" %*

默认情况下，安装程序以批处理模式运行 `Batch mode
<https://djangocms-installer.readthedocs.io/en/latest/usage.html#batch-mode-default>`_, 
并使用一些默认值设置新项目。

稍后，您可能希望自己管理其中的一些，在这种情况下，您需要以向导模式运行它。
默认的批处理模式是设置一个只使用英语的项目，这对于本教程的目的来说已经足够了。
当然，您可以随时编辑新项目的settings.py文件来更改或添加站点语言或修改其他设置。

安装程序为您创建一个admin用户，用户名/密码admin/admin。


启动 runserver
======================

::

    python manage.py runserver

在浏览器中打开http://localhost:8000/，应该邀请您登录，然后创建一个新页面。

.. image:: /introduction/images/welcome.png
   :alt: a django CMS home page
   :width: 400
   :align: center

祝贺您，现在已经安装了一个功能齐全的CMS。

如果您需要随时登录，请在URL中添加?edit并单击Return。这将启用工具栏，您可以从这里登录和管理您的网站。

如果您还不熟悉django CMS，您可以花几分钟的时间为用户浏览django CMS教程的基础知识。
