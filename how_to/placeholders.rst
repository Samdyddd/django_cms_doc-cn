.. _placeholders_outside_cms:

#######################################
How to use placeholders outside the CMS 如何在CMS之外使用占位符
#######################################

占位符是django CMS用来在模板中呈现用户可编辑内容(插件)的特殊模型字段。
也就是说，它是一个地方，用户可以添加文本，视频或任何其他插件到一个网页，
使用相同的前端编辑作为CMS页面。

占位符可以看作CMSPlugin实例的容器，并且可以在CMS之外使用占位符字段在定制应用程序中使用。

通过在自定义模型上定义一个(或多个)PlaceholderField，您可以充分利用CMSPlugin的强大功能。

***********
Get started
***********

你需要在你想要使用的模型上定义一个PlaceholderField::

    from django.db import models
    from cms.models.fields import PlaceholderField

    class MyModel(models.Model):
        # your fields
        my_placeholder = PlaceholderField('placeholder_name')
        # your methods


PlaceholderField有一个必需的参数，一个字符串``slotname``。

``slotname`` 用于模板中，用于确定占位符的插件应该出现在页面的哪个位置，
以及在占位符配置``CMS_PLACEHOLDER_CONF``中，``CMS_PLACEHOLDER_CONF``确定可以将哪些插件插入到这个占位符中。

您还可以对``slotname``使用可调用的参数，如:

    from django.db import models
    from cms.models.fields import PlaceholderField

    def my_placeholder_slotname(instance):
        return 'placeholder_name'

    class MyModel(models.Model):
        # your fields
        my_placeholder = PlaceholderField(my_placeholder_slotname)
        # your methods

.. warning::

    出于安全原因，占位符字段的related_name不能使用``“+”``来禁止;这允许cms正确地检查权限。
    尝试这样做将会引发``ValueError``错误。

.. note::

    如果将PlaceholderField添加到现有模型中，则只有在保存相关实例之后，才能在frontend编辑器中看到占位符。

Admin Integration 管理集成
=================

.. versionchanged:: 3.0

您的带有PlaceholderFields的模型仍然可以在管理中编辑。
但是，其中的任何placeholderfield都只能从前端进行编辑。
PlaceholderFields不能出现在任何字段集、字段、表单或其他ModelAdmin字段的定义属性中。

要在应用程序的管理中为带有``PlaceholderField``的模型提供管理支持，
您需要使用mixin PlaceholderAdminMixin :class:`~cms.admin.placeholderadmin.PlaceholderAdminMixin`
和ModelAdmin :class:`~django.contrib.admin.ModelAdmin。
注意，PlaceholderAdminMixin必须在类定义的``ModelAdmin``之前::

    from django.contrib import admin
    from cms.admin.placeholderadmin import PlaceholderAdminMixin
    from myapp.models import MyModel

    class MyModelAdmin(PlaceholderAdminMixin, admin.ModelAdmin):
        pass

    admin.site.register(MyModel, MyModelAdmin)

I18N Placeholders 国际化占位符
=================

 :class:`~cms.admin.placeholderadmin.PlaceholderAdminMixin` 支持多种语言，并将显示语言选项卡。
如果扩展从 ``PlaceholderAdminMixin`` 派生的模型管理类并覆盖``change_form_template``，请查看
``admin/placeholders/placeholder/change_form.html`` 何显示语言选项卡。

如果需要翻译其他字段，django CMS支持`django-hvad`_。如果使用 ``TranslatableModel``模型，
请确保在翻译的字段中不包含占位符字段:

    class MultilingualExample1(TranslatableModel):
        translations = TranslatedFields(
            title=models.CharField('title', max_length=255),
            description=models.CharField('description', max_length=255),
        )
        placeholder_1 = PlaceholderField('placeholder_1')

        def __unicode__(self):
            return self.title

在向管理站点注册模型时，务必结合 hvad's ``TranslatableAdmin`` and :class:`~cms.admin.placeholderadmin.PlaceholderAdminMixin` ::

    from cms.admin.placeholderadmin import PlaceholderAdminMixin
    from django.contrib import admin
    from hvad.admin import TranslatableAdmin
    from myapp.models import MultilingualExample1

    class MultilingualModelAdmin(TranslatableAdmin, PlaceholderAdminMixin, admin.ModelAdmin):
        pass

    admin.site.register(MultilingualExample1, MultilingualModelAdmin)

Templates
=========

要在模板中呈现占位符，可以使用:mod:`~cms.templatetags.cms_tags`模板标记库中的:ttag:`render_placeholder`标记:

.. code-block:: html+django

    {% load cms_tags %}

    {% render_placeholder mymodel_instance.my_placeholder "640" %}

The :ttag:`render_placeholder` 标签接受以下参数:

* :class:`~cms.models.fields.PlaceholderField` instance
* ``width`` parameter for context sensitive plugins (optional)
* ``language`` keyword plus ``language-code`` string to render content in the
  specified language (optional)

呈现占位符字段的视图必须在上下文中返回请求对象:class:`request <django.http.HttpRequest>`。
这通常是在Django应用程序中使用:class:`~django.template.RequestContext`实现的:

    from django.shortcuts import get_object_or_404, render

    def my_model_detail(request, id):
        object = get_object_or_404(MyModel, id=id)
        return render(request, 'my_model_detail.html', {
            'object': object,
        })

如果你想渲染来自特定语言的插件，你可以这样使用标签:

.. code-block:: html+django

    {% load cms_tags %}

    {% render_placeholder mymodel_instance.my_placeholder language 'en' %}

*******************************
Adding content to a placeholder
*******************************

.. versionchanged:: 3.0

占位符可以通过访问显示模型的页面(放置:ttag:`render_placeholder`标记的位置)，
然后将``?edit``附加到页面的URL中，从而从前端编辑占位符。

这将使前端编辑器顶部横幅出现(如果需要，将要求您登录)。

一旦进入前端编辑模式，应用程序的``PlaceholderFields``接口的工作方式将与CMS页面的工作方式大致相同，
只是需要切换结构和内容模式等等

一般Django模型没有自动的draft/live功能，所以只要add/edit它们，内容就会立即更新。

Options
=======

如果需要将 ``?edit`` 可以将(say, ``?admin_on``) 自定义字符串 ,
 在``settings.py`` 中的``CMS_TOOLBAR_URL__EDIT_ON`` 变量设置为``"admin_on"``.

你也可以改变其他URLs与类似的设置::

* ``?edit_off`` (``CMS_TOOLBAR_URL__EDIT_OFF``)
* ``?build`` (``CMS_TOOLBAR_URL__BUILD``)
* ``?toolbar_off`` (``CMS_TOOLBAR_URL__DISABLE``)

更改这些设置时，请小心，因为您可能无意中替换了系统中保留的字符串 (such as ``?page``).
我们建议您为这个选项使用安全的惟一字符串
(such as ``secret_admin`` or ``company_name``).

.. _placeholder_object_permissions:

Permissions 权限
===========

To be able to edit a placeholder user must be a ``staff`` member and needs either edit permissions
on the model that contains the :class:`~cms.models.fields.PlaceholderField`, or permissions for
that specific instance of that model. Required permissions for edit actions are:
要能够编辑占位符用户，必须是工作人员``staff`` member ，
并且需要包含占位符字段:class:`~cms.models.fields.PlaceholderField`的模型的编辑权限，
或者该模型的特定实例的权限。编辑操作所需的权限是:

* to ``add``: require ``add`` **or** ``change`` 对相关模型或实例的权限.
* to ``change``: require ``add`` **or** ``change`` 对相关模型或实例的权限。
* to ``delete``: require ``add`` **or** ``change`` **or** ``delete`` 对相关模型或实例的权限。

使用这种逻辑，可以更改模型实例但不能添加新模型实例的用户将能够向现有模型的实例添加一些占位符或插件。

模型权限通常通过缺省Django auth应用程序及其管理界面添加。对象级权限可以通过编写自定义身份验证后端来处理，
如django文档中所述 `django docs
<https://docs.djangoproject.com/en/stable/topics/auth/customizing/#handling-object-permissions>`_

例如，如果有一个``UserProfile``模型包含一个``PlaceholderField``，那么自定义后端可以引用一个``has_perm``方法(在模型上)，
该方法仅根据用户的``UserProfile``对象将所有权限授予当前用户::

    def has_perm(self, user_obj, perm, obj=None):
        if not user_obj.is_staff:
            return False
        if isinstance(obj, UserProfile): 
            if user_obj.get_profile()==obj:
                return True
        return False


.. _django-hvad: https://github.com/kristianoellegaard/django-hvad
