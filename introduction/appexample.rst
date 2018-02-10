################################################################################
应用程序
################################################################################
.. contents::
  :local:
  :depth: 3


GAChain平台应用程序是一个由合约、数据库表和接口组成的系统，它执行特定的功能，提供专用服务。该应用程序不是一个自主的程序模块，而唯一与应用程序元素统一起来的是特定函数的性能和数据交换。应用程序的界限不一定是严格定义的，因为它的元素可以同时在多个应用程序中使用。

以下是应用程序的主要功能：

1. 跳转到显示页面: 

* 从数据库表（ *DBFind* 函数）获取信息；
* 形成表单字段，供用户输入新数据；
* 调用合约的按钮（*Button* 函数）。

2. 合约调用:

* 表单字段中的数据传递给合约的 ``data`` 部分；
* ``conditions`` 部分检查用户调用该合约的权限以及该页面接收的数据有效性，如果由于某种原因无法执行合约，则在屏幕上显示一条消息（``error``， ``warning`` 或 ``info``），用户将留在同一页面上；
* ``action`` 部分从数据库表接收附加数据（ *DBFind* 函数）并将数据记录到数据库中（使用 *DBUpdate* 和 *DBInsert* 函数）。

3. 成功执行合约后，用户将跳转到特定页面，该页面的名称在调用合约的 *Button* 函数的 *Page* 参数中生成，并且 *PageParams* 中列出的参数会传递到新页面。

一个页面可以有多个按钮来调用各种合约。调用合约和跳转页面的按钮，可以通过显示用户界面的对象数据的行中构建。在这种情况下，对象标识符作为按钮参数，可用于跳转到与对象相关的页面（例如，对象模块编辑）。
  
=========================
数据库表和数据存储
=========================

应用程序使用数据库表，分为两种类型：

1. 存储关于对象（人员，组织，财产等）的结构化数据的大型数组的数据库表；
2. 文档数据库表，存储由特定应用程序（流程类型、阶段、签名等）实现流程的当前状态，或关于其当前操作的数据，如通知、消息、投票和事务的记录。

通常情况下，数据库表包含最新的信息，这意味着只要接收到新的数据，数据库表的对象模块信息就会更新。该文档表的工作是基于完全不同的原则。由于智语言（G Language）和乾语言（Chain Language）的 *DBFind* 函数只能从一个数据库表中请求信息，这意味着它们不支持 *JOIN* 。因此有必要在存储文档的数据库表记录详尽信息（ID，名称，图片）。例如，当从存储消息的数据库表中请求消息时，我们不仅需要接收消息 ``id``，还有在用户界面上显示所需的所有信息，包括用户名和头像（userpic）。保存后，文档表中的数据不应该被修改。值得注意的是，数据库表中数据的非标准化并不仅仅与 *JOIN* 使用技术限制有关，而是与区块链的概念规定为时间数据库有关，该数据库是为存储完整历史事件而设计的。这意味着在任何情况下都不能修改保存在数据库表中的已签名的文档（例如，消息），即使管理员在用户的注册表中更改其名称也不可以（这在关系数据库中是不可避免的）。

=========================
导航
=========================
用户可以在应用程序使用模块之间进行切换，该模块在软件客户端中显示为选项卡或使用级联菜单。点击部分选项卡后，用户将跳转到该模块的主页面，该页面可以通过管理工具定义。
 
应用程序中的导航大部分都是使用菜单执行（每个页面都有一个这样的的菜单）。页面之间的切换可以使用链接（ *LinkPage* ）或按钮（ *Button* ）点击来实现。在这两种情况下，参数 *PageParams* 都可以传递给 *Page* 目标页面。在成功执行合约后，也可以调用新页面，这需要调用合约 *Button* 函数的参数中包含要跳转到页面（*Page*）的名称和传递 *PageParams* 的参数。 
	
多用户应用程序中的导航可以使用 *Notification* 和 *Roles* 应用程序来组织，这些应用程序默认安装在每个生态系统中。该 *Notification* 应用程序允许在Molis软件客户端中向特定用户或用户角色显示消息。 *Notification* 消息由标题和指向页面的链接组成。此外，参数可以使用 *PageParams* 以格式传递到目标页面，格式由 *LinkPage* 和 *Button* 函数提供。有时，一个目标页面，例如用户需要做出某种决定，只能从通知信息中访问，因为菜单或其他页面没有链接到此目标页面。通知是由合约 *Notifications_Single_Send* 或 *Notifications_Roles_Send* 组成，它从结束应用程序某个功能阶段的合约中调用。在用户收到通知、打开目标页面并执行所需的操作(执行合约)之后，应该通过调用 *Notifications_Single_Close* 或 *Notifications_Roles_Finish* 合约来停止通知。通知类型和其状态列表可在 *Notification* 应用程序中查询。

=========================
界面
=========================
页面结构使用 ``If(Condition){Body } .ElseIf(Condition){Body} .Else{Body}`` 条件语句构成，当调用 *LinkPage* 或 *Button* 函数时，可使用 *PageParam* 的参数发送到页面。如果某些代码片段需要在许多页面中使用，应将其记录到页面块中。这些块可以使用 *Include* 函数使其包含在一个页面中。

=========================
变量，列名称和语言资源
=========================		
统一对变量（页面和合约）、接口页面表单字段标识符、表列命名和语言资源标签，可以大大加快应用程序的开发速度，使程序代码更易于阅读。假设我们想要将参数从接口页面传递给合约，如果合约的数据部分中用户名的变量名称与传递给接口页面的用户名的字段名称相对应，则不需要在 *Button* 函数的 *params* 参数中指定 ``username=username``。变量名和列名称相同可以更容易地使用 *DBInsert* 和 *DBUpdate* 函数，例如， ``DBUpdate("member", $id, "username",$username)``。变量名和语言资源标签相同可以更容易地显示接口表的列名称，例如， ``Table(mysrc,"ID=id,$username$=username")``。

=========================
访问权限
=========================
应用程序最重要的元素是对其系统资源访问权限的管理。这些访问权限可以在以下几个层次上建立：

1. 当前用户调用特定合约的权限。可以在合约的 ``Contions`` 部分中使用 ``if`` 语句中的逻辑表达式或嵌套合约来设置权限。例如，*MainConditions* 或 *RoleConditions* 定义了主要的权限或用户角色权限；
2. 当前用户调用合约来更改表列中的值或向表中添加行的权限。可以使用 *ContractConditions* 函数在表列的 *Permissions* 字段和表编辑页上的 *Permissions/Insert* 字段中设置权限；
3. 仅允许特定合约更改表列中的值或向表中添加行的权限。合约名称应在 *ContractAccess* 函数的参数中指定，该参数应写入表列的 *Persance* 字段，以及表编辑页上的 *Permistions/Insert* 字段；
4. 允许编辑应用程序元素(合约、页面、菜单和页面块)的权限。可以在元素编辑器中的 *ChangeContainments* 字段中设置权限。使用 *ContractConditions* 函数，需要将检查当前用户权限的合约名称作为参数传递给该函数。

=========================
应用程序示例：SendTokens
=========================
应用程序从一个用户帐户向另一个用户帐户发送通证（Token）。关于帐户上通证（Token）数量的信息存储在 *key* 表( *value* 列)中，默认情况下，这些表安装在生态系统中。这个例子意味着通证（Token）已经分配给用户帐户。

系统合约
-----------------
该应用程序的主要合约是 *TokenTransfer* ，它具有更改 *key* 表 *value* 列中值的唯一权限。为了激活该权限，我们在 *AUNT* 列的 *PersERS* 字段中编写 ``ContractAccess("TokenTransfer")`` 函数。此后，任何带有通证（Token）的操作都只能通过调用 *TokenTransfer* 合约来执行。
		
为了防止帐户持有人在不知道的情况下，他的 *TokenTransfer* 合约在另一合约中执行。 *TokenTransfer* 应当是一项有确认机制的合约，其 ``data`` 部分应包含 ``Signature string "optional hidden"`` ，并且确认参数应在Molis管理工具中的 *Contracts With Confirmation* 页面中设置，其中包括：在弹出信息窗口中向用户显示的文本和参数(详情见 *Contracts With Confirmation* 一节)。

.. code:: js

    contract TokenTransfer {
    data {
        Amount money
        Sender_AccountId int
        Recipient_AccountId int
        Signature string "optional hidden"
    }
    conditions {
        //check the sender
        $sender = DBFind("keys").Where("id=$", $Sender_AccountId)
        if(Len($sender) == 0){
            error Sprintf("Sender %s is invalid", $Sender_AccountId)
        }
        $vals_sender = $sender[0]
    
        //check the recipient
        $recipient = DBFind("keys").Where("id=$", $Recipient_AccountId)
        if(Len($recipient) == 0){
            error Sprintf("Recipient %s is invalid", $Recipient_AccountId)
        }
        $vals_recipient = $recipient[0]
    
        //check amount
        if $Amount == 0 {
            error "Amount is zero"
        }
    
        //check balance
        var sender_balance money
        sender_balance = Money($vals_sender["amount"])
        if $Amount > sender_balance {
            error Sprintf("Money is not enough %v < %v", sender_balance, $Amount)
        }
    }
    action {
        DBUpdate("keys", $Sender_AccountId, "-amount", $Amount)
        DBUpdate("keys", $Recipient_AccountId, "+amount", $Amount)
    }
    }
    		
在 *TokenTransfer* 合约的条件部分进行下列检查：交易所涉及的帐户是否存在，要转移的通证（Token）数量是否非零，交易额应该小于或等于发款人帐户的余额。行为部分会对发款人和收款人帐户的 *amount* 列中的值进行修改。

		
表单发送通证（Token）
----------------------------------
该表单发送通证（Token）包含输入交易金额和收款人地址的字段。

.. code:: js

    Div(Class: panel panel-default){
      Form(){ 
        Div(Class: list-group-item text-center){
          Span(Class: h3, Body: LangRes(SendTokens))  
        }
        Div(Class: list-group-item){
          Div(Class: row df f-valign){
            Div(Class: col-md-3 mt-sm text-right){
              Label(For: Recipient_Account){
                Span(Body: LangRes(Recipient_Account))
              }
            }
            Div(Class: col-md-9 mb-sm text-left){
              Input(Name: Recipient_Account, Type: text, Placeholder: "xxxx-xxxx-xxxx-xxxx") 
            } 
          }
          Div(Class: row df f-valign){
            Div(Class: col-md-3 mt-sm text-right){
              Label(For: Amount){
                Span(Body: LangRes(Amount))
              }
            }
            Div(Class: col-md-9 mc-sm text-left){
              Input(Name: Amount, Type: text, Placeholder: "0", Value: "5000000")
            } 
          }
        }
        Div(Class: panel-footer clearfix){
          Div(Class: pull-right){
            Button(Body: LangRes(send), Contract: SendTokens, Class: btn btn-default)
          }
        }
      }
    }      

我们可以使用 *Button* 函数直接调用 *TokenTransfer* 转移合约并传递当前用户(发款人)的帐户地址，但是为了证明合约工作确认机制，我们将创建一个用户调解合约 *SendTokens* 。需要注意的是，由于合约的数据部分中的数据名称和接口表单字段的名称是相同的，所以我们不需要在 *Button* 函数中指定 *Params* 参数。

该表单可以放在软件客户端的任何页面上。合约执行结束后，用户将停留在当前页面上，因为我们没有在 *Button* 函数中指定目标页面。

自定义合约
-----------------
*TokenTransfer* 合约被定义为具有确认性机制的合约，这就是为什么为了从另一个合约调用它，我们需要将签名字符串 ``signature:TokenTransfer`` 放在自定义合约的 ``Data`` 部分里。
*SendTokens* 合约的条件部分会检查帐户的可用性，行为部分调用 *TokenTransfer* 合约并传递参数。

.. code:: js

    contract SendTokens {
        data {
            Amount money
            Recipient_Account string
            Signature string "signature:TokenTransfer"
        }
    
        conditions {
            $recipient = AddressToId($Recipient_Account)
            if $recipient == 0 {
                error Sprintf("Recipient %s is invalid", $Recipient_Account)
            }
        }
    
        action {
            TokenTransfer("Amount,Sender_AccountId,Recipient_AccountId,Signature", $Amount, $key_id, $recipient, $Signature)
        }
    }


