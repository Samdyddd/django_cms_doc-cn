.. versionadded:: 3.0

########################
How to manage Page Types 如何管理页面类型
########################

**Page Types** 使内容编辑器更容易从预定义的类型创建页面。

这些示例包含插件等内容，这些插件将被复制到新创建的页面，而类型则保持不变。


*******************
Creating Page Types
*******************

首先，您需要以通常的方式创建一个新页面;这将成为新页面类型的模板。

使用此页面作为模板，添加示例内容和插件，直到得到满意的结果。

一旦准备好，从页面菜单中选择Save as Page Type…并给它一个合适的名称。
不要担心让它变得完美，你可以继续改变它的内容和设置。

这将创建一个新的页面类型，并通过**Add page**命令和create wizard对话框使其可用。

.. image:: /contributing/images/add-page-type.png
   :alt: Creating a page type

如果您不想要或不需要用于创建新页面类型的原始页面，您可以简单地删除它。


*******************
Managing Page Types
*******************

当您将页面保存为页面类型时，它将放在页面类型节点下的页面列表中。

此节点的行为与常规页面不同:

- 它们不能公开访问
- *Page Types*中列出的所有页面将在“Page Types”下拉菜单中呈现。

还有一种快速创建新页面类型的方法:简单地将现有页面拖到*page Types*节点，然后它将成为新的页面类型。


*********************
Selecting a Page Type 选择页面类型
*********************

现在，您可以在创建新页面时选择页面类型。您将发现一个名为*Page Type*的下拉菜单，您可以从中选择新页面的类型。

.. image:: /contributing/images/select-page-type.png
   :alt: Selecting a page type
