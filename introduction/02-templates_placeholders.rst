########################
Templates & Placeholders
########################

在本教程中，我们将介绍占位符，还将展示如何使自己的HTML模板cms就绪。


*********
Templates
*********

您可以使用HTML模板来定制网站的外观，定义占位符来标记托管内容的部分，并使用特殊标记来生成菜单等等。

您可以使用不同的布局或内置组件定义多个模板，并根据需要为每个页面选择它们。
一个页面的模板可以随时切换到另一个模板。

你可以在该目录中找到站点模板 ``mysite/templates``.

默认情况下，站点中的页面将使用fullwidth.html模板，
这是项目settings.py CMS_TEMPLATES元组中列出的第一个模板:

..  code-block:: python
    :emphasize-lines: 3

    CMS_TEMPLATES = (
        ## Customize this
        ('fullwidth.html', 'Fullwidth'),
        ('sidebar_left.html', 'Sidebar Left'),
        ('sidebar_right.html', 'Sidebar Right')
    )


************
Placeholders
************

占位符是在HTML模板中定义节的一种简单方法，当页面呈现时，这些节将被来自数据库的内容填充。
这些内容是使用django CMS的前端编辑机制(使用django模板标记)编辑的。

``fullwidth.html`` 包含一个占位符, ``{% placeholder "content" %}``.

可以看到 ``{% load cms_tags %}`` 在那个文件中 - ``cms_tags`` 是所需的模板标记库。

If you're not already familiar with Django template tags, you can find out more in the `Django documentation
<https://docs.djangoproject.com/en/dev/topics/templates/>`_.

Add a couple of new placeholders to ``fullwidth.html``, ``{% placeholder "feature" %}`` and ``{%
placeholder "splashbox" %}`` inside the ``{% block content %}`` section. For example:

.. code-block:: html+django
   :emphasize-lines: 2,4

    {% block content %}
        {% placeholder "feature" %}
        {% placeholder "content" %}
        {% placeholder "splashbox" %}
    {% endblock content %}

If you switch to *Structure* mode, you'll see the new placeholders available for use.

.. image:: /introduction/images/new-placeholder.png
   :alt: the new 'splashbox' placeholder
   :align: center


*******************
Static Placeholders
*******************

The content of the placeholders we've encountered so far is different for
every page. Sometimes though you'll want to have a section on your website
which should be the same on every single page, such as a footer block.

You *could* hard-code your footer into the template, but it would be nicer to be
able to manage it through the CMS. This is what **static placeholders** are for.

Static placeholders are an easy way to display the same content on multiple
locations on your website. Static placeholders act almost like normal
placeholders, except for the fact that once a static placeholder is created and
you added content to it, it will be saved globally. Even when you remove the
static placeholders from a template, you can reuse them later.

So let's add a footer to all our pages. Since we want our footer on every
single page, we should add it to our **base template**
(``mysite/templates/base.html``). Place it near the end of the HTML ``<body>`` element:

.. code-block:: html+django
   :emphasize-lines: 1-3

        <footer>
          {% static_placeholder 'footer' %}
        </footer>


        {% render_block "js" %}
    </body>

Save the template and return to your browser. Refresh any page in Structure mode, and you'll
see the new static placeholder.

.. image:: /introduction/images/static-placeholder.png
   :alt: a static placeholder
   :align: center

..  note::

    To reduce clutter in the interface, the plugins in static placeholders are hidden by default.
    Click or tap on the name of the static placeholder to reveal/hide them.

If you add some content to the new static placeholder in the usual way, you'll see that it
appears on your site's other pages too.


***************
Rendering Menus
***************

In order to render the CMS's menu in your template you can use the :doc:`show_menu
</reference/navigation>` tag.

Any template that uses ``show_menu`` must load the CMS's ``menu_tags`` library
first:

.. code-block:: html+django

    {% load menu_tags %}

The menu we use in ``mysite/templates/base.html`` is:

.. code-block:: html+django

    <ul class="nav navbar-nav">
        {% show_menu 0 100 100 100 "menu.html" %}
    </ul>

The options control the levels of the site hierarchy that are displayed in the menu tree - but you don't need to worry about exactly what they do at this stage.

Next we'll look at :ref:`integrating_applications`.
