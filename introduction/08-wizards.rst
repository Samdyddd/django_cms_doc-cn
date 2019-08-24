########################
Content creation wizards
########################

内容创建向导允许您在自己的应用程序中使用工具栏的Create按钮。
它打开一个简单的对话框，其中包含创建新项目所需的基本字段。

django CMS使用它来创建页面，但是您可以向其中添加自己的模型。

在polls_cms_integration应用程序中，添加一个forms.py文件:

    from django import forms

    from polls.models import Poll


    class PollWizardForm(forms.ModelForm):
        class Meta:
            model = Poll
            exclude = []

Then add a ``cms_wizards.py`` file, containing::

    from cms.wizards.wizard_base import Wizard
    from cms.wizards.wizard_pool import wizard_pool

    from polls_cms_integration.forms import PollWizardForm


    class PollWizard(Wizard):
        pass

    poll_wizard = PollWizard(
        title="Poll",
        weight=200,  # determines the ordering of wizards in the Create dialog
        form=PollWizardForm,
        description="Create a new Poll",
    )

    wizard_pool.register(poll_wizard)

刷新民意调查页面，点击工具栏中的``Create``按钮，向导对话框将打开，为您提供一个用于创建民意调查的新向导。

.. note::

    同样，这个特殊的例子只是为了说明。
    在轮询的情况下，它的多个问题通过外键与之关联，我们真的希望能够同时编辑这些问题。

    这将需要比本教程范围内可能的更复杂的表单和处理。
