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

如果您还不熟悉Django模板标记，您可以在Django文档中找到更多信息。 `Django documentation
<https://docs.djangoproject.com/en/dev/topics/templates/>`_.

添加连个新的占位符到 ``fullwidth.html``, ``{% placeholder "feature" %}`` and ``{%
placeholder "splashbox" %}`` inside the ``{% block content %}`` section. 例如:

.. code-block:: html+django
   :emphasize-lines: 2,4

    {% block content %}
        {% placeholder "feature" %}
        {% placeholder "content" %}
        {% placeholder "splashbox" %}
    {% endblock content %}

如果切换到结构模式，您将看到可以使用的新占位符。

.. image:: /introduction/images/new-placeholder.png
   :alt: the new 'splashbox' placeholder
   :align: center


*******************
Static Placeholders
*******************

：到目前为止，我们遇到的占位符的内容对于每个页面都是不同的。
虽然有时候你想在你的网站上有一个每一页都应该相同的部分，比如页脚。

您可以将页脚硬编码到模板中，但是能够通过CMS管理页脚会更好。这就是静态占位符的作用。

静态占位符是在网站的多个位置显示相同内容的简单方法。
静态占位符的行为几乎与普通占位符一样，除了一个事实，即一旦创建了一个静态占位符并向其添加了内容，它将被全局保存。
即使从模板中删除静态占位符，也可以稍后重用它们

让我们给所有页面添加一个页脚。因为我们希望每个页面都有页脚，所以应该将它添加到基本模板
(``mysite/templates/base.html``). 将其放在HTML``<body>`` 元素的末尾:

.. code-block:: html+django
   :emphasize-lines: 1-3

        <footer>
          {% static_placeholder 'footer' %}
        </footer>


        {% render_block "js" %}
    </body>

保存模板并返回到浏览器。在结构模式下刷新任何页面，您将看到新的静态占位符。

.. image:: /introduction/images/static-placeholder.png
   :alt: a static placeholder
   :align: center

..  note::

    为了减少界面的混乱，默认情况下，静态占位符中的插件是隐藏的。单击或轻击静态占位符的名称以显示/隐藏它们。

如果您按照通常的方式向新的静态占位符添加一些内容，您将看到它也会出现在站点的其他页面上。


***************
Rendering Menus
***************

为了在模板中呈现CMS的菜单，可以使用  :doc:`show_menu
</reference/navigation>` 标签。

任何使用show_menu的模板都必须先加载CMS的menu_tags库:

.. code-block:: html+django

    {% load menu_tags %}

我们在mysite/templates/base.html中使用的菜单是:

.. code-block:: html+django

    <ul class="nav navbar-nav">
        {% show_menu 0 100 100 100 "menu.html" %}
    </ul>

这些选项控制了在菜单树中显示的站点层次结构的级别——但是在这个阶段您不需要担心它们具体做了什么。

Next we'll look at :ref:`integrating_applications`.
