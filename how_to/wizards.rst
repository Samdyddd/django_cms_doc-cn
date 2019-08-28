.. versionadded:: 3.2

.. _wizard_how_to:

#########################################
How to implement content creation wizards
如何实现内容创建向导
#########################################

django CMS提供了一个为内容编辑器创建向导的框架。
它们为常见任务提供了简化的工作流。
django CMS页面向导已经存在，但是您可以非常容易地为其他内容类型创建自己的页面向导。

********************************
Create a content-creation wizard
创建一个内容创建向导
********************************

为自己的模块创建CMS内容创建向导相当容易。

首先，在模块的根级创建一个名为``forms.py``的文件来创建表单:

    # my_apps/forms.py

    from django import forms

    class MyAppWizardForm(forms.ModelForm):
        class Meta:
            model = MyApp
            exclude = []

现在在根级别创建另一个名为``cms_wizards.py``的文件。在该文件中，导入 ``Wizard`` 如下:

    from cms.wizards.wizard_base import Wizard
    from cms.wizards.wizard_pool import wizard_pool

然后，只需子类向导，实例化它，然后注册它。如果你为MyApp做这个，它可能是这样的:

    # my_apps/cms_wizards.py

    from cms.wizards.wizard_base import Wizard
    from cms.wizards.wizard_pool import wizard_pool

    from .forms import MyAppWizardForm

    class MyAppWizard(Wizard):
        pass

    my_app_wizard = MyAppWizard(
        title="New MyApp",
        weight=200,
        form=MyAppWizardForm,
        description="Create a new MyApp instance",
    )

    wizard_pool.register(my_app_wizard)

.. note::

    如果您的模型没有定义``get_absolute_url``函数，那么您的向导将需要一个``get_success_url``方法。

    ..  code-block:: python

        class MyAppWizard(Wizard):

            def get_success_url(self, obj, **kwargs):
                """
                This should return the URL of the created object, «obj».
                """
                if 'language' in kwargs:
                    with force_language(kwargs['language']):
                        url = obj.get_absolute_url()
                else:
                    url = obj.get_absolute_url()

                return url

That's it!

.. note::

    模块名``cms_wizard``是特殊的，因为在您的项目的Python路径中，
    任何这样命名的模块都将自动加载，从而触发在其中找到的任何向导的注册。
    向导可以在其他模块中声明和注册，但它们可能不会自动加载。

The above example is using a ``ModelForm``, but you can also use ``forms.Form``.
In this case, you **must** provide the model class as another keyword argument
when you instantiate the Wizard object.

上面的示例使用的是``ModelForm``，但是您也可以使用``forms.Form``。
在这种情况下，在实例化向导对象时，必须将model类作为另一个关键字参数提供。

For example::

    # my_apps/forms.py

    from django import forms

    class MyAppWizardForm(forms.Form):
        name = forms.CharField()


    # my_apps/cms_wizards.py

    from cms.wizards.wizard_base import Wizard
    from cms.wizards.wizard_pool import wizard_pool

    from .forms import MyAppWizardForm
    from .models import MyApp

    class MyAppWizard(Wizard):
        pass

    my_app_wizard = MyAppWizard(
        title="New MyApp",
        weight=200,
        form=MyAppWizardForm,
        model=MyApp,
        description="Create a new MyApp instance",
    )

    wizard_pool.register(my_app_wizard)

必须子类化 ``cms.wizards.wizard_base.Wizard`` 向导使用它。这是因为每个向导的惟一性由它的类和模块名决定。

See the :ref:`Reference section on wizards <wizard_reference>` for technical details of the wizards
API.
有关向导API的技术细节，请参阅向导参考部分。