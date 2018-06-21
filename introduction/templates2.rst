###########
用户界面
###########

.. contents::
  :local:
  :depth: 2

***********
创建界面
***********
Molis软件客户端是一个集成开发环境，使用 ``React`` 框架开发，其中包括一个界面编辑器和一个虚拟界面编辑器。界面是应用程序的重要组成部分，提供从数据库表中检索和显示数据，创建用于接收用户输入数据的表单，将数据传递给合约以及在应用程序页面之间的导航。界面类似于合约，存储在区块链中，这样可以在软件客户端加载时防止伪造。

界面模版引擎
==============================
界面元素（页面和菜单）在模板引擎的验证节点上形成，该模板在Moils软件客户端的页面编辑器里开发。所有界面均使用 ``乾语言（Chain Language）`` 编写。可以使用内置 ``API`` 从网络上的节点请求页面。模板引擎对这种API请求的回复，发送的内容不是 ``HTML`` 页面，而是根据模板结构形成树的 ``HTML`` 标签组成的 ``JSON`` 代码。为了测试，可以使用包含要处理的模板名称的参数使用 ``POST`` 请求发送到 ``api/v2/content``。

创建界面模板
==============================
Molis客户端有专门的编辑器创建和编辑界面，在Molis的管理工具的界面中可以找到。编辑器提供：

- 自动补全和代码高亮显示；
- 菜单用来显示页面；
- 编辑页面的菜单；
- 配置权限以编辑页面（通过在 ``ContractConditions`` 函数中指定具有权限的合约名称，或通过在更改条件字段中直接指示访问权限）；
- 启动一个可视化界面编辑器；
- 页面预览。

可视化界面编辑器
-----------------------------
可视化界面编辑器无需使用 ``乾语言（Chain Language）`` 的界面源代码即可创建页面。编辑器允许使用拖曳方式来设置页面上表单元素和文本的位置，以及配置页面块的大小。编辑器提供了一套用于显示典型数据模型的即用型块：包含标题，表单和信息面板的面板。在创建页面设计之后，可以在页面编辑器中添加程序逻辑（接收数据和条件结构）。我们计划将来创建一个更全面的可视化界面编辑器。

样式的使用
-----------------------------
默认情况下，界面页面使用 ``Angular Bootstrap Angle`` 样式显示。用户可以创建自己的样式。样式存储使用生态系统配置表的特殊样式表参数实现。

页面区块
-----------------------------
在多个界面上使用典型的代码片段，可以使用 ``Insert`` 命令创建页面区块并将其嵌入到界面代码中。这些区块可以在 ``Molis`` 的管理部分的界面上创建和编辑。对于区块，可以像定义页面一样编辑权限。

语言资源编辑器
-----------------------------
Molis软件客户端包含一个使用 ``乾语言（Chain Language）`` 模板语言的特殊功能的界面定位机制 - ``LangRes`` ，该语言在软件端或浏览器页面端，可代替对应文本行的语言资源标签。语法 ``$lable`` 的 *$* 可以用来代替 ``LangRes`` 函数。用 ``智语言（G Language）`` 编写的 ``LangRes`` 函数执行合约，会弹出信息转换的窗口。

语言资源可以在Molis软件客户端的管理工具的语言资源部分创建和编辑。一个语言资源由一个标签（名称）和该名称的其他语言版本组成，该语言会指示相应的双字符语言标识符（CN,HK,EN,FR,JP等）。

在Molis管理工具的 *数据表** 模块的语言包里，添加和更改语言资源包的方法都是一样。

********************************************************************************
乾语言（Chain Language）模版语言
********************************************************************************

``乾语言（Chain Language）`` 函数提供了以下操作：

- 从数据库中检索值： ``DBFind`` ；
- 从数据库中检索数据的表示形式；
- 赋值和显示变量的值，使用数据的操作： ``SetVar,GetVar,Data``；
- 显示和比较日期/时间值： ``DateTime,Now,CmpTime``；
- 用各种用户数据输入字段建立表单： ``form, imageInput, Input, RadioGroup, select``；
- 显示错误消息验证表单字段中的数据： ``Validate,InputErr``；
- 显示导航元素： ``AddToolButton,LinkPage,Button``；
- 调用合约： ``Button``；
- 创建 ``HTML`` 页面布局元素 - 具有指定 ``CSS`` 类选项的各种容器： ``Div,P,Span`` 等；
- 将图像嵌入页面并上传图像： ``Image`` 和 ``ImageInput``；
- 有条件地显示页面布局片段： ``If,ElseIf,Else``；
- 创建多级菜单；
- 界面模块本地化。

模板语言乾语言（Chain Language）概述
============================================================
页面模板语言是一种函数语言，它允许使用 ``FuncName(parameters)`` 调用函数，以及将函数嵌套到彼此中。可以指定参数而不用引号，不必要的参数可以丢弃。

.. code:: js

      Text MyFunc(parameter number 1, parameter number 2) another text.
      MyFunc(parameter 1,,,parameter 4)
      
如果参数包含逗号，则应该用引号（后引号或双引号）括起来。如果函数只能有一个参数，可以在其中使用逗号而不用引号。如果参数有不成对的右括号，则应使用引号。

.. code:: js

      MyFunc("parameter number 1, the second part of first paremeter")
      MyFunc(`parameter number 1, the second part of first paremeter`)
      
如果将参数放在引号中，但参数本身包含引号，则可以使用单引号 ``''`` 或者使用反引号 `````。
      
.. code:: js

      MyFunc("parameter number 1, ""the second part of first"" paremeter")
      MyFunc(`parameter number 1, "the second part of first" paremeter`)
      
在函数描述中，每个参数都有一个特定的名字。你可以调用函数并按照声明的顺序指定参数，也可以按任意顺序指定任何一组参数： ``"Parameter_name：Parameter_value"`` 。这种方法可以安全地添加新的函数参数，而不会破坏与当前模板的兼容性。例如，所有这些调用在语言使用上都是正确的，它被描述为 ``MyFunc(Class，value，body)`` 。

.. code:: js

      MyFunc(myclass, This is value, Div(divclass, This is paragraph.))
      MyFunc(Body: Div(divclass, This is paragraph.))
      MyFunc(myclass, Body: Div(divclass, This is paragraph.))
      MyFunc(Value: This is value, Body: 
           Div(divclass, This is paragraph.)
      )
      MyFunc(myclass, Value without Body)
      
函数可以返回文本，生成 ``HTML`` 元素（例如 ``"Input"`` ），或者使用嵌套的 ``HTML`` 元素 ``(Div,P,Span)`` 创建 ``HTML`` 元素。在后一种情况下，应使用具有预定义名称 ``Body`` 的参数来定义嵌套元素。例如，嵌套在另一个 ``div`` 中的两个 ``div`` 可以如下所示：

.. code:: js

      Div(Body:
         Div(class1, This is the first div.)
         Div(class2, This is the second div.)
      )
      
要定义 ``Body`` 参数中描述的嵌套元素，可以使用以下表示形式： ``MyFunc(...){...}`` 。嵌套元素应在大括号中指定。

.. code:: js

      Div(){
         Div(class1){
            P(This is the first div.)
            Div(class2){
                Span(This is the second div.)
            }
         }
      }
      
如果需要连续多次指定相同的函数，则可以使用 ``.`` 而不是每次都写入函数名称。例如，以下几行是相等的：
     
.. code:: js

     Span(Item 1)Span(Item 2)Span(Item 3)
     Span(Item 1).(Item 2).(Item 3)
     
该语言允许使用 **SetVar** 函数分配变量。要使用 ``#varname#`` 替换变量的值。

.. code:: js

     SetVar(name, My Name)
     Span(Your name: #name#)
     
要替换生态系统的语言资源，可以使用 ``$langres$`` ，其中 ``langres`` 是语言源的名称。

.. code:: js

     Span($yourname$: #name#)
     
以下是预定义变量

* ``#key_id#`` - 当前用户帐户ID；
* ``#ecosystem_id#`` - 当前的生态系统ID。

使用PageParams将参数传递给页面
-----------------------------------------------------------
有许多函数支持 ``PageParams`` 参数，当跳转到一个新页面时，这个函数可以传递参数。例如， ``PageParams:"param1 = value1,param2 = value2"`` 。参数值既可以是简单的字符串，也可以是具有替换值的行。当参数传递给页面时，会创建带有参数名称的变量。例如: ``#param1#`` 和 ``#param2#`` 。

* ``PageParams: "hello=world"`` - 页面会收到以 ``world`` 为值的 ``hello`` 参数
* ``PageParams: "hello=#world#"`` - 页面会收到 ``hello`` 参数，其值为 ``world`` 变量。

另外， ``Val`` 函数允许跳转到指定的表单获取数据。在这种情况下，

* ``PageParams: "hello=Val(world)"`` - 页面会收到 ``hello`` 参数，其中包含 ``world`` 元素的值。


调用合约
-----------------------------
``乾语言（Chain Language）`` 通过单击表单中的按钮（ *Button* 函数）来实现合约调用。一旦启动这个事件，用户在界面表单的字段中输入的数据被传递给合约，如果表单字段的名称对应调用合约的数据部分中的变量的名称，数据将自动传输。该按钮函数允许打开一个模态窗口，用于合约执行用户验证（Alert），以及在成功执行合约之后跳转到指定页面，并将某些参数传递给此页面。 

********************************************************************************
乾语言（Chain Language）的函数操作
********************************************************************************

变量操作
==============================
GetVar(Name)
------------------------------
如果存在的情况下，此函数返回当前变量的值，或者如果未定义具有此名称的变量，则返回空字符串。当请求编辑的树时才创建具有 ``getvar`` 名称元素。  ``GetVar(varname)`` 和 ``#varname#`` 之间的区别是，如果 ``varname`` 不存在， ``GetVar`` 将返回一个空字符串，而 ``#varname#`` 将被解释为一个字符串值。

* *Name* - 变量名称

.. code:: js

     If(GetVar(name)){#name#}.Else{Name is unknown}
      
SetVar(Name, Value)
------------------------------
将一个值赋给一个 ``Name`` 变量。

* *Name* - 变量的名称；
* *Value* - 变量的值，可以包含对另一个变量的引用。

.. code:: js

     SetVar(name, John Smith).(out, I am #name#)
     Span(#out#)      

界面导航
==============================     
AddToolButton(Title, Icon, Page, PageParams)
------------------------------------------------------------
向按钮面板添加一个按钮。创建 **AddToolButton** 元素。

* *Title* - 按钮标题；
* *Icon* - 按钮的图标；
* *Page* - 跳转的页面名称；
* *PageParams* - 页面的参数。

.. code:: js

      AddToolButton(Help, help, help_page) 
      
Button(Body, Page, Class, Contract, Params, PageParams)[.CompositeCOntract(Name,Data)] [.Alert(Text,ConfirmButton,CancelButton,Icon)] [.Popup(Width, Header)] [.Style(Style)]
------------------------------------------------------------------------------------------------------------------------------------------------------

创建一个 **Button** HTML元素。这个元素创建一个按钮，通过点击可执行合约或者跳转至指定页面。

* *Body* - 文本信息或者元素，用于输入按钮的名称；
* *Page* - 要跳转的页面名称；
* *Class* - 样式类；
* *Contract* - 执行的合约名称；
* *Params* - 传递给合约的数值列表。默认情况下，合约参数（``data``）的值是从具有相似名称标识符（id）的HTML元素（例如，Input）获得的。如果元素标识符合约参数的名称不同，则应使用 ``contractField1 = idname1,contractField2 = idname2`` 格式中的分配。该参数作为目标对象 ``{field1:idname1,field2:idname2}`` 返回给 ``attr`` ；
* *PageParams* - 跳转到页面的参数，格式：``contractField1 = idname1，contractField2 = idname2`` 。在这种情况下，目标页面上会创建参数名称为  ``#contractField1#`` 和 ``#contractField2#`` 的变量，并为其分配指定的值（请参阅 *使用PageParams将参数传递给页面* 部分）。
**CompositeContract** - 连接按钮的附加合约。
     * *Name* - 合约名称；
     * *Data* - JSON数组，传递给合约所需的参数。

**Alert** - 弹窗显示消息。

* *Text* - 消息文本；
* *ConfirmButton* - 确认按钮标题；
* *CancelButton* - 取消按钮标题；
* *Icon* - 按钮图标。

**Popup** - 显示模态窗口

* *Header* - 窗口的标题；
* *Width* - 窗口宽度百分比，取值范围为1到100。

**Style** - 用于指定CSS样式。

* *Style* - css样式。

.. code:: js

      Button(Submit, default_page).CompisiteContract(NewPage, [{"Name":"Name of Page"},{"Value":"Span(Test)"}])
      Button(Submit, default_page, mybtn_class).Alert(Alert message)
      Button(Submit, default_page, mybtn_class).Popup(Header: message, Width: 50)
      Button(Contract: MyContract, Body:My Contract, Class: myclass, Params:"Name=myid,Id=i10,Value")
	  
LinkPage(Body, Page, Class, PageParams) [.Style(Style)]
------------------------------------------------------------
创建一个 **LinkPage** 元素 - 一个页面的链接。
 
* *Body* - 子文本或元素；
* *Page* - 页面重定向到；
* *Class* - 这个按钮的类；
* *PageParams* - 重定向参数。

**Style** - 指定CSS样式

* *Style* - css样式。

.. code:: js

      LinkPage(My Page, default_page, mybtn_class)

数据操作
==============================
And (Parameters)
------------------------------
该函数返回 **and** 逻辑运算的执行结果，括号中列出所有参数，并以逗号分隔。 如果参数值等于空字符串( ``""`` ),零或 *false*，则参数值为 ``"false"`` 。 在所有其他情况下，参数值是 ``"true"`` 。 如果为 ``true`` ，则函数返回 ``1``,否则返回 ``0`` 。仅当请求编辑的树模块时才创建名 ``And`` 元素。

.. code:: js

      If(And(#myval1#,#myval2#), Span(OK))

Calculate(Exp, Type, Prec)
------------------------------------------------------------
该函数返回 **Exp** 参数中传递算术表达式的结果。使用以下操作：+，-，x，/和（）。

* **Exp** - 算术表达式。包含数字和 *#name#* 变量；
* **Type** - 结果数据类型: ``int, float, money`` 。如果没有指定，类型默认是 *float* 。如果有小数点的数字，或者在其他所有情况下 *int* ；
* **Prec** -  *float* 和 *money* 类型指定后的有效位数点；

例如： ``Calculate( Exp: (342278783438+5000)*(#val#-932780000), Type: money, Prec:18 )``， ``Calculate(10000-(34+5)*#val#)``， ``Calculate("((10+#val#-45)*3.0-10)/4.5 + #val#", Prec: 4)``。

CmpTime(Time1, Time2)
------------------------------
此函数比较两个时间值的格式相同（最好是标准格式 - YYYY-MM-DD HH：MM：SS，但是可以使用任何格式，前提是序列从几年到几秒）。返回：

* **-1** - Time1 < Time2； 
* **0** - Time1 = Time2；
* **1** - Time1 > Time2。

.. code:: js

     If(CmpTime(#time1#, #time2#)<0){...}
     
DateTime(DateTime, Format)
------------------------------
此函数以指定的格式显示时间和日期。

 *  *DateTime* - 时间和日期标准格式 ``2006-01-02T15:04:05``；
 *  *Format* -  格式: ``YY`` 2位年份格式, ``YYYY`` 4位年份格式, ``MM`` - 月份, ``DD`` - 天, ``HH`` - 小时, ``MM`` - 分钟, ``SS`` – 秒。 例如： ``YY/MM/DD HH:MM``。 如果未指定格式，则将使用 *languages* 表中设置的 *timeformat* 参数值。 如果这个参数不存在，则将使用 ``"YYYY-MM-DD HH:MI:SS"`` 格式。
 
 .. code:: js

    DateTime(2017-11-07T17:51:08)
    DateTime(#mytime#,HH:MI DD.MM.YYYY)

Now(Format, Interval) 
------------------------------
该函数以指定的格式返回当前时间，默认情况下是 ``UNIX`` 格式（ ``1970年1月1日`` 以来经过的秒数）。 如果请求的时间格式是 *datetime*，则日期和时间显示为 ``"YYYY-MM-DD HH:MI:SS"`` 。 间隔可以在第二个参数中指定（例如，*+5天*）。

* *Format* - 输出的格式为 ``YYYY, MM, DD, HH, MI, SS`` 或 *datetime*；
* *Interval* - 向后或向前的时间偏移。

.. code:: js

       Now()
       Now(DD.MM.YYYY HH:MM)
       Now(datetime,-3 hours)

Or(parameters)
------------------------------
此函数返回 **IF** 逻辑运算结果，其中所有参数在括号中指定，并以逗号分隔。 如果参数值等于空字符串(``""``)， ``0`` 或 ``false`` ，则参数值为 ``"false"`` 。 在所有其他情况下，参数值被认为是 ``"true"`` 。 该函数在其他情况下返回 ``1`` 或 ``0`` 。元素 **Or** 仅在请求编辑的树时才创建。

.. code:: js

      If(Or(#myval1#,#myval2#), Span(OK))

数据显示
==============================
Code(Text)
------------------------------
创建一个 **Code** 元素来显示指定的代码。

	
* *Text* - 源代码。

.. code:: js

      Code( P(This is the first line.
          Span(This is the second line.))
      )  

Chart(Type, Source, FieldLabel, FieldValue, Colors)
------------------------------------------------------------------------------------------
创建HTML图表。

 * *Type* - 图表类型；
 * *Source* - 数据源的名称，例如 *DBFind* 命令；
 * *FieldLabel* - 标题字段的名称；
 * *FieldValue* - 值字段的名称；
 * *Colors* - 颜色列表。

.. code:: 

      Data(mysrc,"name,count"){
          John Silver,10
          "Mark, Smith",20
          "Unknown ""Person""",30
      }
      Chart(Type: "bar", Source: mysrc, FieldLabel: "name", FieldValue: "count", Colors: "red, green")
	  
ForList(Source, Body)
------------------------------
以 *Body* 中设置的模板格式显示来自 *Source* 数据源的元素列表，并创建 **ForList** 元素。

* *Source* - 数据源来自 *DBFind* 或 *Data* 函数；
* *Body* - 插入元素的模板。

.. code:: js

      ForList(mysrc){Span(#name#)}
      
Image(Src,Alt,Class) [.Style(Style)]
------------------------------------------------------------
创建一个 **Image** 元素的标签。
 
* *Src* - 图片源，文件或 ``data:...``；
* *Alt* - 替代图片的文字；
* *Сlass* - 类名。

.. code:: js

    Image(\images\myphoto.jpg)    
    
MenuGroup(Title, Body, Icon) 
------------------------------
在菜单中形成一个嵌套子菜单并返回 **MenuGroup** 元素。 在用语言资源替换之前，*name* 参数也将返回 *Title* 的值。

* *Title* - 菜单项名称；
* *Body* - 菜单组中的子菜单；
* *Icon* - 图标。

.. code:: js

      MenuGroup(My Menu){
          MenuItem(Interface, sys-interface)
          MenuItem(Dahsboard, dashboard_default)
      }
      
MenuItem(Title, Page, Params, Icon, Vde) 
------------------------------------------------------------
创建一个菜单项并返回 **MenuItem** 元素。

* *Title* - 菜单项名称；
* *Page* - 页面重定向到；
* *Params* - 参数以 ``var:value`` 格式传递给页面，用逗号分隔；
* *Icon* - 图标；
* *Vde* -  定义在虚拟生态系统过渡的参数。如果 ``Vde:true`` ，则链接重定向到VDE。如果 ``Vde:false`` ，则链接重定向到区块链。如果没有指定参数，则根据菜单的加载位置来定义。

.. code:: js

       MenuItem(Interface, interface)
       
Table(Source, Columns) [.Style(Style)]
------------------------------------------------------------
创建一个 ``HTML`` 的 ``table`` 元素。

* *Source* - 数据源名称，例如，在 **DBFind** 命令中；
* *Columns* - 标题和相应的列名称，如下所示： ``Title1=column1,Title2=column2``。

**Style** - 指定CSS样式

* *Style* - css样式。

.. code:: js

      DBFind(mytable, mysrc)
      Table(mysrc,"ID=id,Name=name")
      
数据接收
==============================
Address (account)
------------------------------
这个函数返回地址数值给出的 ``1234-5678 -...- 7990`` 格式的帐号地址。 如果没有指定地址，则将当前用户地址作为参数。

.. code:: js

      Span(Your wallet: Address(#account#))

Data(Source,Columns,Data) [.Custom(Column,Body)]
------------------------------------------------------------
创建 **Data** 元素并填充指定的数据放入 *Source* ，然后在 *Table* 和其他命令resivieng *Source* 中指定输入的数据。 列名的顺序对应于 *data* 条目的值。
 
* *Source* - 数据源名称。 你可以指定任何名称，稍后必须将其包含在其他命令（例如 *Table* ）中作为数据源；
* *Columns* - 列的列表；
* *Data* - 每行一个数据条目，用逗号分隔。 数据应该与 *Columns* 中设置的顺序相同，Entry值可以用双引号括起来。 如果你需要在文本中使用引号，请使用双引号；
* **Custom** - 允许为数据分配计算列表。 例如，你可以为按钮和其他页面布局元素指定一个模板。 可以分配多个计算列表。 通常，这些字段被分配用于输出到 *Table* 和其他使用接收数据的命令。
 
  * *Column* - 列名称。 应该分配一个独特的名字；
  * *Body* - 一个代码片段。 你可以使用 ``#columnname#`` 从这个条目的其他列中获取值，并在这个代码片段中使用它们。

.. code:: js

    Data(mysrc,"id,name"){
	"1",John Silver
	2,"Mark, Smith"
	3,"Unknown ""Person"""
     }.Custom(link){Button(Body: View, Class: btn btn-link, Page: user, PageParams: "id=#id#"}    

DBFind(table, Source) [.Columns(columns)] [.Where(conditions)] [.WhereId(id)] [.Order(name)] [.Limit(limit)] [.Offset(offset)] [.Ecosystem(id)] [.Custom(Column,Body)][.Vars(Prefix)]
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
创建 **dbfind** 元素，使用 *table* 表中的数据填充它，并将其放到 *Source* 结构中。 *Source* 结构可以用于 *Table* 和其他接收 *Source* 的命令作为输入数据。 *data* 中的记录顺序应该与列名称的顺序相对应。

* **Name** - 表名称；
* **Source** - 任意数据源名称；
* **Columns** - 要返回的列的列表。如果未指定，则返回所有列。如果有JSON类型的列，您可以使用以下语法来处理记录字段: **columnname->fieldname**。在本例中，生成的列名将是 **columnname.fieldname**；
* **Where** - 数据搜索条件，例如： ``.where(name = '#myval#')``。如果有JSON类型的列，您可以使用以下语法来处理记录字段: **columnname->fieldname**；
* **WhereId** - 按ID搜索。 例如： ``.WhereId(1)``；
* **Order** - 字段排序；
* **Limit** - 返回的行数， ``Default value = 25, maximum value = 250``；
* **Offset** - 返回行的偏移量；
* **Ecosystem** - 生态系统ID。 默认情况下，数据是从当前生态系统的指定表中获取的；
* **Custom** - 允许为数据分配计算列。例如，你可以为按钮和其他页面布局元素指定一个模板。你可以分配任意数量的计算列。通常，这些字段被分配用于输出到 *Table* 和其他使用接收数据的命令：
 
  * *Column* - 列名称。 应该分配一个独特的名字；
  * *Body* - 一个代码片段。 你可以使用 ``#columnname#`` 从此条目中的其他列中获取值，并在此代码片段中使用它们；
* **Vars** - 该函数将从该查询中获取数据库表中的值，并生成一组变量。 指定此函数时， *Limit* 参数自动变为1，并且只返回一条记录。

  * *Prefix* - 前缀函数用于为变量生成名称，并将结果行的值保存到该变量中：变量格式为 ``#prefix_id#，#prefix_name#`` ，其中列名称紧跟下划线符号。如果有包含JSON字段的列，则结果变量将是以下格式 ``#prefix_columnname_field#``。

.. code:: 

    DBFind(parameters,myparam)
    DBFind(parameters,myparam).Columns(name,value).Where(name='money')
    DBFind(parameters,myparam).Custom(myid){Strong(#id#)}.Custom(myname){
       Strong(Em(#name#))Div(myclass, #company#)
    }

EcosysParam(Name, Index, Source) 
------------------------------------------------------------
该函数从当前生态系统的参数表中获取参数值。 如果有结果名称的语言资源，则会进行相应的翻译。
 
* *Name* - 值名称；
* *Index* - 例如，如果 ``gender = male,female`` ,那么 ``EcosysParam(gender,2)`` 将会在以下情况下被指定为一个由逗号分隔的元素列表 返回 *famle*；
 
* *Source* - 你可以把接收由逗号分隔的参数值作为 *data* 对象。 之后，你将能够将此列表指定为 *Table* 和 *Select* 的数据源。如果你指定了这个参数，那么函数会返回一个列表作为 *Data* 对象，而不是一个单独的值。


.. code:: js

     Address(EcosysParam(founder_account))
     EcosysParam(gender, Source: mygender)
 
     EcosysParam(Name: gender_list, Source: src_gender)
     Select(Name: gender, Source: src_gender, NameColumn: name, ValueColumn: id)
     
LangRes(Name, Lang)
------------------------------
返回指定的语言资源。如果要求编辑树，则返回 ``$langres$`` 元素。

* *Name* - 语言资源的名称；
* *Lang* - 默认情况下，返回是 *Accept-Language* 请求中定义的语言。你可以指定自己的双字符语言标识符。可以指定lcid标识符，例如： *en-US、en-GB* 。在这种情况下，如果没有找到所请求的值，例如，对于 *en-Us*，那么将在 *en* 中查找语言资源。

.. code:: js

      LangRes(name)
      LangRes(myres, fr)     

SysParam(Name) 
------------------------------
显示 ``system_parameters`` 表中系统参数的值。

* *Name* - 参数名称。

.. code:: js

     Address(SysParam(founder_account))

数据格式元素
============================== 
Div(Class, Body) [.Style(Style)]
------------------------------------------------------------
创建一个 *HTML* 的 ``div`` 元素。

* *Class* - 这个 *div* 元素的类；
* *Body* - 子元素。

**Style** - 用于指定css样式。

* *Style* - CSS样式。

.. code:: js

      Div(class1 class2, This is a paragraph.)
      
Em(Body, Class)
------------------------------
创建一个 *HTML* 的 ``em`` 元素

* *Body* -  子标签或文本；
* *Class* -  ``em`` 元素的类名。

.. code:: js

      This is an Em(important news).
      
P(Body, Class)
------------------------------
创建一个 ``P`` 标签

* *Body* - 子标签或文本，
* *Class* - ``p`` 元素的类名。

**Style** - 指定CSS样式，

* *Style* - CSS样式。

.. code:: js

      P(This is the first line.
        This is the second line.)
	
SetTitle(Title)
------------------------------
设置页面标题。 元素 **SetTitle** 将被创建。

* *Title* - 页面标题。

.. code:: js

     SetTitle(My page)	
	
Label(Body, Class, For) [.Style(Style)]
------------------------------------------------------------
创建一个 **Label** HTML元素。

* *Body* - 文本或者 ``HTML`` 元素；
* *Class* -  类名称；
* *For* - 属性下的值。

**Style** - 设置css样式。

* *Style* - CSS样式。

.. code:: js

      Label(The first item).	
	
Span(Body, Class) [.Style(Style)]
------------------------------------------------------------
创建一个 ``span`` 标签。

* *Body* - 文本或者 ``HTML`` 元素；
* *Class* -  ``span`` 标签的类名。

**Style** - 设置CSS样式。

* *Style* - css样式。

.. code:: js

      This is Span(the first item, myclass1).
      
Strong(Body, Class)
------------------------------
创建一个 ``Strong`` 的标签。

* *Body* - 文本或者 ``HTML`` 元素；
* *Class* - 类名。

.. code:: js

      This is Strong(the first item, myclass1).
      
表单元素
==============================      
Form(Class, Body) [.Style(Style)]
------------------------------------------------------------
创建一个 ``form`` 表单。

* *Class* - 类名；
* *Body* - 子元素。

**Style** - 设置样式。

* *Style* - css样式。

.. code:: js

      Form(class1 class2, Input(myid))
      
ImageInput(Name, Width, Ratio, Format) 
------------------------------------------------------------
这个函数为图片上传创建一个 **imageinput** 元素。 在第三个参数中，你可以指定要应用的图像高度或高宽比：*1/2*，*2/1*，*3/4* 等。默认宽度为 ``100像素`` ，宽高比为 ``1/1`` 。

* *Name* - 元素名称；
* *Width* - 裁剪图像的宽度；
* *Ratio* - 宽高比；
* *Format* - 上传图片的格式。

.. code:: js

   ImageInput(avatar, 100, 2/1)    
   
Input(Name,Class,Placeholder,Type,Value) [.Validate(validation parameters)] [.Style(Style)]
------------------------------------------------------------------------------------------------------------------------
创建一个 ``input`` 的表单元素。

* *Name* - 元素名称；
* *Class* -  ``input`` 表单的类名；
* *Placeholder* -  ``input`` 表单的 ``Placeholder`` ；
* *Type* - *input* 表单类型；
* *Value* - 元素值。

**Validate** - 参数验证。

**Style** - 设置样式。

* *Style* - css样式。

.. code:: js

      Input(Name: name, Type: text, Placeholder: Enter your name)
      Input(Name: num, Type: text).Validate(minLength: 6, maxLength: 20)

InputErr(Name,validation errors)]
------------------------------------------------------------
用验证错误文本创建一个 **inputerr** 元素。

* *Name* - 对应的 **Input** 元素的名称。

.. code:: js

      InputErr(Name: name, 
          minLength: Value is too short, 
          maxLength: The length of the value must be less than 20 characters)
	  

RadioGroup(Name, Source, NameColumn, ValueColumn, Value, Class) [.Validate(validation parameters)] [.Style(Style)]
------------------------------------------------------------------------------------------------------------------------------------
创建一个 **RadioGroup** 元素。

* *Name* - 元素名称；
* *Source* - 数据源，例如：通过 *DBFind* 或 *Data* 函数获取；
* *NameColumn* - 列名要使用元素名称的来源；
* *ValueColumn* - 列名称使用元素值的来源。 此参数不能使用使用自定义创建的列；
* *Value* - 默认值；
* *Class* - 元素的类。

**Validate** - 参数验证。

**Style** - 设置样式。

* *Style* - css样式。

.. code:: js

      DBFind(mytable, mysrc)
      RadioGroup(mysrc, name)	  
      
Select(Name, Source, NameColumn, ValueColumn, Value, Class) [.Validate(validation parameters)] [.Style(Style)]
--------------------------------------------------------------------------------------------------------------------------
创建一个 ``select`` 的下拉列表。

* *Name* - 元素名称；
* *Source* - 数据源，例如：通过 *DBFind* 或 *Data* 函数获取；
* *NameColumn* - 元素名称的列表；
* *ValueColumn* - 元素值的列表。自定义创建列表时不应该指定该参数；
* *Value* - 默认值；
* *Class* - 元素的类名。

**Validate** - 参数验证。

**Style** - 设置样式。

* *Style* - CSS样式。

.. code:: js

      DBFind(mytable, mysrc)
      Select(mysrc, name) 
      
代码操作
=========================
If(Condition){ Body } [.ElseIf(Condition){ Body }] [.Else{ Body }]
--------------------------------------------------------------------
条件表达式为 ``true`` 返回 ``If`` 语句块，否则返回 ``ElseIf`` 或者 ``Else`` 语句块。

* *Condition* - 如果条件等于 ``空字符串 0 或 false`` ，则认为条件未满足。否则，条件被视为正确；

* *Body* - 子元素。

.. code:: js

      If(#value#){
         Span(Value)
      }.ElseIf(#value2#){Span(Value 2)
      }.ElseIf(#value3#){Span(Value 3)}.Else{
         Span(Nothing)
      }
   
Include(Name)
------------------------------
该命令在页面的代码中插入模板，名称为 *Name* 。

* *Name* - 模板的名称。

.. code:: js

      Div(myclass, Include(mywidget))
      
************************************************
移动端样式
************************************************

排版
==============================

标题
------------------------------

* ``h1`` ... ``h6``

重点类名
------------------------------

* ``.text-muted``
* ``.text-primary``
* ``.text-success``
* ``.text-info``
* ``.text-warning``
* ``.text-danger``

颜色
------------------------------

* ``.bg-danger-dark``
* ``.bg-danger``
* ``.bg-danger-light``
* ``.bg-info-dark``
* ``.bg-info``
* ``.bg-info-light``
* ``.bg-primary-dark``
* ``.bg-primary``
* ``.bg-primary-light``
* ``.bg-success-dark``
* ``.bg-success``
* ``.bg-success-light``
* ``.bg-warning-dark``
* ``.bg-warning``
* ``.bg-warning-light``
* ``.bg-gray-darker``
* ``.bg-gray-dark``
* ``.bg-gray``
* ``.bg-gray-light``
* ``.bg-gray-lighter``

栅格系统
==============================
* ``.row``
* ``.row.row-table``
* ``.col-xs-1`` ... ``.col-xs-12`` 仅在父级具有 ``.row.row-table`` 类时才有效。

面板
==============================

* ``.panel``
* ``.panel.panel-heading``
* ``.panel.panel-body``
* ``.panel.panel-footer``

表单
==============================

* ``.form-control``

按钮
==============================

* ``.btn.btn-default``
* ``.btn.btn-link``
* ``.btn.btn-primary``
* ``.btn.btn-success``
* ``.btn.btn-info``
* ``.btn.btn-warning``
* ``.btn.btn-danger``

图标
==============================

所有图标来自FontAwesome: ``fa fa-<icon-name></icon-name>``。

所有图标来自SimpleLineIcons: ``icon-<icon-name>``。
   
      
