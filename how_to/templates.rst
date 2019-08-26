##########################
How to work with templates 如何使用模板
##########################

应用程序可以通过混合cms模板标签和普通django模板语言来重用cms模板。


static_placeholder
------------------

外部应用程序使用的模板中不能使用普通占位符Plain :ttag:`placeholder`，请使用:ttag:`static_placeholder`。

.. _page_template:

CMS_TEMPLATE
------------
.. versionadded:: 3.0

``CMS_TEMPLATE`` 是上下文中可用的上下文变量;它包含使用apphooks的CMS页面和应用程序的模板路径，
以及默认模板(i.e.: the first template in :setting:`CMS_TEMPLATES`) 用于non-CMS管理的url。

在应用程序模板中的``extends``模板标记中使用它来获取当前页面模板，这非常有用。

Example: cms template

.. code-block:: html+django

    {% load cms_tags %}
    <html>
        <body>
        {% cms_toolbar %}
        {% block main %}
        {% placeholder "main" %}
        {% endblock main %}
        </body>
    </html>


Example: application template

.. code-block:: html+django

    {% extends CMS_TEMPLATE %}
    {% load cms_tags %}
    {% block main %}
    {% for item in object_list %}
        {{ item }}
    {% endfor %}
    {% static_placeholder "sidebar" %}
    {% endblock main %}

``CMS_TEMPLATE`` 记忆cms模板的路径，以便应用程序模板可以动态导入它。


render_model
------------
.. versionadded:: 3.0

:ttag:`render_model` 允许通过重用django CMS前端编辑器从前端编辑django模型。