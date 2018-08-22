################################################################################
智能合约
################################################################################
.. contents::
  :local:
  :depth: 2

智能合约（以下简称“合约”）是应用程序的基本元素，它通过用户或另一个合约从用户界面发起、执行单个操作，该操作通常是在数据库表中进行新增，修改或者查询操作。在应用程序中使用数据的所有操作都是通过合约系统形成的，通过数据库表或合约主体中的函数调用相互交互。

政务链中的合约系统是使用原生的（由政务链团队开发的）图灵脚本语言（称为智语言（G Language））编写，并编译成字节码。该语言包括一组函数、操作符和结造，可用于实现数据处理算法和数据库操作。

合约可以编辑，但是如果在合约编辑权限中设置 ``false`` 则无法编辑。区块链中的数据操作由合约的最新（当前）版本执行。合约更改的所有历史记录都存储在区块链中，可从软件客户端查看。

***********
合约的结构
***********
合约由 ``contract`` 关键字声明，对应当前合约的名称。合约结构用大括号括起来。所有的合约都由三个部分组成：

1. **data** - 声明输入数据（变量名称及其类型）；
2. **conditions** - 验证输入数据的正确性；
3. **action** - 合约执行的操作。

合约结构:

.. code:: js

  contract MyContract {
      //数据部分
      data {
          FromId address
          ToId   address
          Amount money

          //此处用来声明输入的数据包括（变量的名称和类型）
      }
      //条件部分
      func conditions {
          ...

          //此处用来验证数据
      }
      //动作部分
      func action {
          ...

          //合约的动作部分，通常是操作数据库表
      }
  }
  

数据部分
==============================
在数据部分中描述合约输入的数据，以及用于接收数据形式的参数。数据是逐行列出的：首先指定变量名（只有变量，但不传递数组），可以通过双引号间隔来指定类型和参数：

* *hidden* - 隐藏元素；
* *optional* - 非必填项；
* *date* - 日期和时间；
* *polymap* - 选择坐标和区域的地图；
* *map* - 具有标记地点能力的地图，如百度地图、谷歌地图；
* *file* - 文件字段；
* *text* - 文本信息；
* *crypt:Field* - 为 ``Field`` 字段中指定的目标创建并加密私钥。如果只指定了crypt，那么将为签署合约的用户创建私钥；
* *address* - 用于输入帐户地址的字段；
* *signature:contractname* - 显示合约名称的行，它需要签名（详见 `合约签名` 部分）。

.. code:: js

  contract my {
    data {
        Name string 
        RequestId address
        Photo bytes "file optional"
        Amount money
        Private bytes "crypt:RequestId"
    }
    ...
  }
    
条件部分
========
本节介绍如何对获取数据的验证。以下命令用于警告错误：``error``, ``warning``, ``info``。它们都会产生一个错误，停止合约操作，但在界面中显示不同的消息：严重错误，警告和信息错误。例如，

.. code:: js

  if fuel == 0 {
        error "fuel cannot be zero!"
  }
  if money < limit {
        warning Sprintf("You don't have enough money: %v < %v", money, limit)
  }
  if idexist > 0 {
        info "You have been already registered"
  }

行为部分
========
行为部分包含合约的主程序代码，用于检索附加数据并将结果值记录到数据库表中。例如，

.. code:: js

	action {
		DBUpdate("keys", $key_id,"-amount", $amount)
		DBUpdate("keys", $recipient,"+amount,pub", $amount, $Pub)
	}

另外，合约还可以包含 **price()** 函数，该函数在执行合约时增加额外费用，以燃料为单位。返回值将被添加到合约执行成本并乘以 `fuel_rate` 。

.. code:: js
	
	contract MyContract {
		action {
			DBUpdate("keys", $key_id,"-amount", $amount)
			DBUpdate("keys", $recipient,"+amount,pub", $amount, $Pub)
		}
		func price() int {
		     return 200
		}
	}

合约变量
========
在数据部分中声明的合约通过 ``$`` 符号的数据名称传递到其他部分，从而实现数据输入。 ``$`` 符号可以用来声明额外的变量；这些变量在当前合约和所有嵌套合约具有全局性。

合约可以访问预定义的变量，这些变量包含关于调用该合约的事务的数据。

* ``$time`` – 交易时间戳，int；
* ``$ecosystem_id`` – 生态系统ID，int；
* ``$block`` – 包含此事务的区块编号，int；
* ``$key_id`` – 签署交易的账户地址；
* ``$block_key_id`` – 生成包含此事务的区块的节点的地址；
* ``$block_time`` – 当前合约的交易的区块生成的时间戳，time;
* ``$original_contract`` - 合约的名称，最初被称为事务处理。如果该变量是空字符串，则意味着在验证条件的过程中调用了该合约。为了检查该合同是否被另一个合同或直接从当前事务中调用，比较 **$original_contract** 和 **$this_contract** 的值。如果它们相等，则意味着从当前事务中调用了合约；
* ``$this_contract`` - 当前执行的合约名称；
* ``$stack`` - 数组类型，包含合约名称，数组的第一个元素为当前调用的合约，数组的最后一个元素是处理交易的原始合约。

预定义变量不仅可以在合约中使用，也可以在权限字段（定义访问应用程序元素的条件）中使用，这些变量用于构建逻辑表达式。当在权限字段中使用时，与区块形成（$time，$block等）相关的变量总是等于零。

预定义变量 ``$result`` 用于从嵌套合约中返回一个值。

.. code:: js

  contract my {
    data {
        Name string 
        Amount money
    }
    func conditions {
        if $Amount <= 0 {
           error "Amount cannot be 0"
        }
        $ownerId = 1232
    }
    func action {
        DBUpdate("mytable", $ownerId, "name,amount", $Name, $Amount - 10 )
        DBUpdate("mytable2", $citizen, "amount", 10 )
    }
  }
  
********************************************************************************
嵌套合约 
********************************************************************************
嵌套合约可以从封闭合约的条件和操作部分调用。 嵌套合约可以直接使用名称后面括号中指定的参数（ ``NameContract(Params)`` ）或使用 ``CallContract`` 函数（使用字符串变量为其传递合约名称）来直接调用合约。

********************************************************************************
合约签名
********************************************************************************
由于合约书写的语言允许执行封闭的合约，所以当用户运行外部合约，签名未被授权的事务，也可能不被发现。这可能导致用户对其未经授权的交易进行签名，比如说资金来自其帐户。


假设有一个合约 ``TokenTransfer`` ：

.. code:: js

    contract TokenTransfer {
        data {
          Recipient int
          Amount    money
        }
        ...
    }

如果在由用户发起的合约中签字 ``TokenTransfer("Recipient,Amount", 12345, 100)`` ，100个通证（Token）将被转移到账户12345。在这种情况下，签署外部合约的用户身份将不会再事务处理中出现。如果 ``TokenTransfer`` 合约在其调用合约时需要额外的用户签名，则可能避免上述情况的发生。步骤如下：

1. 在 ``TokenTransfer`` 合约的数据部分添加一个名为 ``Signature`` 的字段，其中包含 ``"optional hidden"`` 参数，由于签名字段中含有签名，因此无需直接调用合约中的附加签名。

.. code:: js

    contract TokenTransfer {
        data {
          Recipient int
          Amount    money
          Signature string "optional hidden"
        }
        ...
    }

2. 在 ``Signature`` 表中（在政务链客户端的签名上）添加包含以下内容的条目：

•	*TokenTransfer* 合约名称；
•	字段名称的值将显示给用户，他们的文字说明；
•	文本信息在确认后显示。
  
在当前的例子中，它将指定两个字段 **Receipient** 和 **Amount**:

* **Title**: 你同意向该接收人发送款项吗？
* **Parameter**: 收件人: Account；
* **Parameter**: 金额: Amount (qGAC)。

现在，如果插入 ``TokenTransfer("Recipient,Amount",12345，100)`` 合约调用，则会显示系统错误 ``"Signature is not defined"`` 。如果按照以下方式调用合约： ``TokenTransfer("Recipient, Amount, Signature", 12345, 100, "xxx...xxxxx")`` ，系统错误将在签名验证时发生。在合约调用时，验证以下信息：``time of the initial transaction, user ID,  the value of the fields specified in the signatures table`` ，从而伪造签名就不会发生。

为了使用户在调用 ``TokenTransfer`` 协议时看到汇款确认，需要添加一个任意名称和类型字符串的字段，并且带有可选参数签名： ``contractname`` 。在调用封闭的 ``TokenTransfer`` 合约之后，你只需转发此参数。还应该牢记的是，外部合约的数据部分还必须描述担保合约的参数（它们可能是隐藏的，但仍会在确认后显示）。例如：

.. code:: js

    contract MyTest {
      data {
          Recipient int "hidden"
          Amount  money
          Signature string "signature:TokenTransfer"
      }
      func action {
          TokenTransfer("Recipient,Amount,Signature",$Recipient,$Amount,$Signature)
      }
    }

在发送 ``MyTest`` 合约时，用户会请求对指定账户转账的额外确认。如果在随附的合约中列出了 ``TokenTransfer(“Recipient,Amount,Signature”,$Recipient, $Amount+10, $Signature)`` 等其他值，将出现无效签名错误。

********************************************************************************
合约编辑
********************************************************************************
合约可以在Molis软件客户端的特定编辑器中创建和编辑。每个新合约都有一个典型的结构，默认情况下有三个部分：数据、条件、行为。合约编辑有助于：

- 编写合约代码的关键词（突出显示智语言（G Language））；
- 格式化合约源代码；
- 将合约绑定到一个帐户，从中执行的付款将被收取；
- 定义编辑合约的权限（通常，通过指定具有特殊功能 ``ContractConditions`` 中规定的权限的合约名称，或通过直接指示更改条件字段中的访问条件）；
- 查看对合约所做更改的历史记录，并选择恢复以前的版本。

********************************************************************************
合约语言智语言（G Language）
********************************************************************************
政务链中的合约是使用原生图灵脚本语言编写，由政务链团队开发，称为智语言（G Language），编译成字节码。该语言包括一组函数，操作符和构造，可用于实现数据处理算法和数据库操作。智语言（G Language）提供：

- 声明不同数据类型的变量，以及简单的和关联的数组： ``var、array、map``；
-  ``if`` 条件语句和 ``while`` 循环结构的使用；
- 从数据库中检索值并将数据记录到数据库 ``DBFind、DBInsert、DBUpdate``；
- 处理合约；
- 变量转换；
- 字符串操作。

基本要素和结构
==============================
数据类型和变量
---------------------------------
数据类型为每个变量定义。通常情况下，数据类型会自动转换。可以使用以下数据类型：

* ``bool`` - 布尔型，可以是 ``true`` 或 ``false`` ；
* ``bytes`` - 字节序列；
* ``int`` - 64位整数；
* ``address`` - 64位无符号整数；
* ``array`` - 任意类型的值的数组；
* ``map`` - 任意数据类型与字符串键值的关联数组；
* ``money`` - 大整数类型的整数，值存储在数据库中，没有小数点，在根据货币配置设置在用户界面中显示值时添加小数点；
* ``float`` - 带有浮点的64位数字；
* ``string`` - 字符串，应该用双引号或后引号定义：“这是一个字符串”或 ``This is a string`` 。

所有标识符，包括变量名称，函数，合约等都区分大小写( ``MyFunc`` 和 ``myFunc`` 是不同的名称)。

变量是用 ``var`` 关键字声明的，接着是变量名称和类型。在大括号内声明的变量应该在同一对大括号内使用。声明时，变量具有默认值：对于bool类型，它是false，对于所有数字类型 - 零值，对于字符串 - 空字符串。变量声明的例子：

.. code:: js

  func myfunc( val int) int {
      var mystr1 mystr2 string, mypar int
      var checked bool
      ...
      if checked {
           var temp int
           ...
      }
  }

数组
---------------------------------
该语言支持两种数组类型：

* ``array`` - 数字索引从零开始的简单数组；
* ``map`` - 带有字符串键的关联数组。

当分配和检索数组元素时，索引应放在方括号中。

.. code:: js

    var myarr array
    var mymap map
    var s string
    
    myarr[0] = 100
    myarr[1] = "This is a line"
    mymap["value"] = 777
    mymap["param"] = "Parameter"

    s = Sprintf("%v, %v, %v", myarr[0] + mymap["value"], myarr[1], mymap["param"])
    // s = 877, This is a line, Parameter 

If 和 While 
---------------------------------
合约语言支持标准条件语句 ``if`` 和 ``while`` 循环，可以在函数和合约中使用。这些语句可以相互嵌套。

关键字必须有一个条件语句。如果条件语句返回一个数字，那么当它的值为0时。例如， ``val == 0`` 等于 ``!val`` ，而 ``val != 0`` 等于 ``val`` 。 ``if`` 语句允许有一个else或多个elif，elif必须包含一个条件。以下比较运算符可用于条件语句：``<,>,>=,<=,==,!=,||和&&`` 。

.. code:: js

    if val > 10 || id != $citizen {
      ...
    } elif val == 5 {
       ...
    } elif val < 0 {
       ...
    } else {
      ...
    }

``while`` 语句旨在实现循环。 一个 ``while`` 语句块将在条件成立时执行。 ``break`` 操作符用于结束块内的循环。要从头开始循环，应该使用 ``continue`` 操作符。

.. code:: js

  while true {
      if i > 100 {
         break
      }
      ...
      if i == 50 {
         continue
      }
      ...
  }

除了条件语句之外，该语言还支持标准算术运算： ``+，-，*，/`` ，字符串和字节类型的变量可以用作条件。在这种情况下，当字符串（字节）的长度大于零时，条件为 ``true``，对于空字符串，则为 ``false``。

函数
---------------------------------	
合约语言的函数使用合约的数据部分接收的数据并执行操作：读取和写入数据库值、转换值类型和建立合约之间的连接。

函数是用 ``func`` 关键字来声明的，接着是函数名和传递给它的参数列表（包含它们的类型），所有的参数都用大括号括起来，并用逗号分开。在大括号之后，应该说明函数返回值的数据类型。该函数应该放在大括号内。如果一个函数没有参数，那么大括号是没有必要的。要从函数返回值，使用 ``return`` 关键字。

.. code:: js

  func myfunc(left int, right int) int {
      return left*right + left - right
  }
  func test int {
      return myfunc(10, 30) + myfunc(20, 50)
  }
  func ooops {
      error "Ooops..."
  }
  
函数不会返回错误，因为所有错误检查都是自动执行的。当在任何函数中生成错误时，合约将停止其操作，并显示带有错误描述的窗口。

未定义数量的参数可以传递给一个函数。要做到这一点，使用 ``...`` ，而不是最后一个参数的类型。在这种情况下，最后一个参数的数据类型将是 ``array`` ，它将包含所有被传递调用的参数和变量。任何类型的变量都可以传递，但应注意，与数据类型不匹配的时候会发生冲突。

.. code:: js

  func sum(out string, values ...) {
      var i, res int
      
      while i < Len(values) {
         res = res + values[i]
         i = i + 1
      }
      Println(out, res)
  }

  func main() {
     sum("Sum:", 10, 20, 30, 40)
  }
  
有一种情况，一个函数有很多参数，但是在调用它的时候我们只需要其中的一部分。可以用下面的方法声明可选参数： ``func myfunc(name string).Param1(param string).Param2(param2 int){...}`` 。你可以按任意顺序只指定你需要的参数： ``myfunc("name").Param2(100)`` 。在函数体中，你可以正常处理这些变量。如果调用未指定扩展参数，则它具有默认值，例如，字符串默认值为空字符串，数字默认值为零。需要注意的是，你可以指定几个扩展参数，并使用 ``...`` ： ``func DBFind(table string).Where(request string,params ...)`` 并调用 ``DBFind("mytable").Where("id > ? and type = ?", myid, 2)`` 。

.. code:: js
 
    func DBFind(table string).Columns(columns string).Where(format string, tail ...)
             .Limit(limit int).Offset(offset int) string  {
       ...
    }
     
    func names() string {
       ...
       return DBFind("table").Columns("name").Where("id=?", 100).Limit(1)
    }

预定义的值
---------------------------------
执行合约时，以下变量可用。

* ``$key_id`` - 签名事务的帐户的数字标识符(int64)；
* ``$ecosystem_id`` - 创建事务的生态系统的标识符；
* ``$type`` 从当前合约被调用的外部合约的标识符；
* ``$time`` - 以Unix格式在事务中指定的时间；
* ``$block`` - 该事务的区块编号；
* ``$block_time`` - 在区块内限定的时间；
* ``$block_key_id`` - 在该区块上签名的节点的数字标识符(int64)；
* ``$auth_token`` 授权通证(Token)，它可以在VDE合约中使用，例如，在使用 ``"HTTPRequest"`` 函数调用合约时。

.. code:: js

	var pars, heads map
	heads["Authorization"] = "Bearer " + $auth_token
	pars["vde"] = "false"
	ret = HTTPRequest("http://localhost:7079/api/v2/node/mycontract", "POST", heads, pars)

需要注意的是，这些变量不仅在合约函数中，而且在其他函数和表达式中也是可用的。例如，在合约，页面和其他对象指定的条件下，与区块有关的 *$time* ，*$block* 变量等于0。

需要从合约中返回的值应该被分配给一个预定义的变量 ``$result``。

数据库中检索值
==============

AppParam(app int, name string) string
-----------------------------------------

该函数返回应用程序参数（ *app_params* 表）指定参数的值。

* *app* - 应用程序ID；
* *name* - 参数名称。

.. code:: js

    Println(AppParam(1, "app_account"))

DBFind(table string) [.Columns(columns string)] [.Where(where string, params ...)] [.WhereId(id int)] [.Order(order string)] [.Limit(limit int)] [.Offset(offset int)] [.Ecosystem(ecosystemid int)] array
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
函数根据指定的请求从数据库表中接收数据。返回的是由 ``map`` 关联数组组成的 ``数组`` 。

* *table* - 数据库表名称；
* *сolumns* - 返回列的列表。如果未指定，则将返回所有列；
* *Where* - 搜索条件,例如： ``.Where("name = 'John'")`` 或 ``.Where("name = ?", "John")``；
* *id* - 按ID搜索，例如， *.WhereId(1)*；
* *order* - 用于排序，默认情况下按 ``ID`` 排序；
* *limit* - 返回值的数量（默认值 = 25, 最大值 = 250）；
* *offset* - 返回值的偏移量；
* *ecosystemid* - 生态系统ID。默认情况下，从当前生态系统的数据表格中获取值。

.. code:: js

   var i int
   ret = DBFind("contracts").Columns("id,value").Where("id> ? and id < ?", 3, 8).Order("id")
   while i < Len(ret) {
       var vals map
       vals = ret[0]
       Println(vals["value"])
       i = i + 1
   }
   
   var ret string
   ret = DBFind("contracts").Columns("id,value").WhereId(10).One("value")
   if ret != nil { 
   	Println(ret) 
   }

DBRow(table string) [.Columns(columns string)] [.Where(where string, params ...)] [.WhereId(id int)] [.Order(order string)] [.Ecosystem(ecosystemid int)] map
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
该函数根据指定的查询返回一个关联数组 ``map`` 和从数据库表中获取的数据。

 * *table* - 数据库表名称；
 * *columns* -要返回的列的列表。如果未指定，则将返回所有列；
 * *Where* - 搜索参数，例如： ``.Where("name = 'John'")`` 或 ``.Where("name = ?", "John")``；
 * *id* - 要返回数据的ID， ``.WhereId(1)``；
 * *order* - 用于分类的字段， 默认情况下，数据按 ``id`` 字段排序，
 * *ecosystemid* - 生态系统ID默认情况下是当前的生态系统ID。
 	
.. code:: js

   var ret map
   ret = DBRow("contracts").Columns("id,value").Where("id = ?", 1)
   Println(map)

DBSelectMetrics(metric string, timeInterval string, aggregateFunc string) array
----------------------------------------------------------------------------------
该函数返回查询 *metrics* 表的 *array* 数组，用于统计对应数量。统计数量每隔100个区块更新一次。

* *metric* - 统计指标名称；

   * ``ecosystem_pages`` - 生态系统界面的数量；
   * ``ecosystem_members`` - 生态系统会员人数；
   * ``ecosystem_tx`` - 生态系统交易数量。

* *timeInterval* - 统计的时间间隔。例如， ``1 Weeks`` 或者 ``30 days``； 
* *aggregateFunc* - 聚合函数，例如， ``max`` 、 ``min`` 、 ``avg``。

.. code:: js

   var rows array
   rows = DBSelectMetrics("ecosystem_tx", "30 days", "avg")
   
   var i int
   while(i < Len(rows)) {
      var row map
      row = rows[i] // row = [map[key:1 value:1463]]
      i = i + 1
   }
    
EcosysParam(name string) string
---------------------------------
该函数返回生态系统（ *parameters* 表）中指定参数的值。

* *name* - 参数的名称。

.. code:: js

    Println( EcosysParam("changing_tables"))

GetBlockHistory(id int64) array 
--------------------------------
该函数返回 *_blocks* 表历史更改记录的数组，每一个数组都包含在上一个变更之前的记录字段，结果列表根据最新更改排序。

* *id* - 更改记录ID。

.. code:: js

    var list array
    var item map
    list = GetBlockHistory(1)
    if Len(list) > 0 {
       item = list[0]
    }

GetContractHistory(id int64) array 
-------------------------------------
该函数返回 *_contracts* 表历史更改记录的数组，每一个数组都包含在上一个变更之前的记录字段，结果列表根据最新更改排序。

* *id* - 更改记录ID。

.. code:: js

    var list array
    var item map
    list = GetContractHistory(1)
    if Len(list) > 0 {
       item = list[0]
    }

GetMenuHistory(id int64) array 
--------------------------------
该函数返回 *_menus* 表历史更改记录的数组，每一个数组都包含在上一个变更之前的记录字段，结果列表根据最新更改排序。

* *id* - 更改记录ID。

.. code:: js

    var list array
    var item map
    list = GetMenuHistory(1)
    if Len(list) > 0 {
       item = list[0]
    }

GetPageHistory(id int64) array 
--------------------------------
该函数返回 *_pages* 表历史更改记录的数组，每一个数组都包含在上一个变更之前的记录字段，结果列表根据最新更改排序。

* *id* - 更改记录ID。

.. code:: js

    var list array
    var item map
    list = GetPageHistory(1)
    if Len(list) > 0 {
       item = list[0]
    }

GetColumnType(table, column string) string
-------------------------------------------
该函数返回 *table* 表中 *column* 列的类型。返回内置类型名称，例如， *text、varchar、number、money、double、bytea、json、datetime、double*。

* *table* - 数据表名称；
* *column* - 列名称。

.. code:: js

    var coltype string
    coltype = GetColumnType("members", "member_name")

GetDataFromXLSX(binId int, line int, count int, sheet int) string
--------------------------------------------------------------------
该函数将数据作为XLSX表中的单元格数组返回。

* *binId* - *binary* 表中XLSX类型的ID；
* *line* - 获取数据的行；
* *count* - 返回的行数；
* *sheet* - XLSX文件中的工作表编号，默认为1。

.. code:: js

    var a array
    a = GetDataFromXLSX(binid, 12, 10, 1)

GetRowsCountXLSX(binId int, sheet int) int
---------------------------------------------
该函数返回XLSX文件中指定工作表上的行数。

* *binId* - *binary* 表中XLSX类型的ID；
* *sheet* - XLSX文件中的工作表编号，默认为1。

.. code:: js

    var count int
    count = GetRowsCountXLSX(binid, 1)

LangRes(appID int64,label string, lang string) string
--------------------------------------------------------
此函数返回lang的语言资源，并指定为双字符代码，例如， ``zh,en,ru`` ，如果所选语言没有语言资源，则结果将以 ``zh`` 返回。

* *appID* - 生态系统ID；
* *label* - 语言资源名称；
* *lang* - 双字符语言代码。

.. code:: js

    warning LangRes(1, "confirm", $Lang)
    error LangRes(2, "problems", "de")

GetBlock(blockID int64) map
------------------------------
该函数返回关于 *blockID* 区块的信息。 返回结果包含：

* *id* - 区块ID；
* *time* - 区块生成时间戳；
* *key_id* - 该区块的节点账户地址。

.. code:: js

   var b map
   b = GetBlock(1)
   Println(b)
                     	
更改数据表的值
==============================
DBInsert(table string, params string, val ...) int
-------------------------------------------------------------------------------------------------
该函数将数据添加到指定的 ``table`` 并返回插入的数据的 ``id`` 。

* *table*  – 数据库表名称；
* *params* - 将列出以逗号分隔的列名称列表，其中 ``val`` 中列出的值将被写入；
* *val* - 参数中列出的列的逗号分隔值列表，值可以是字符串或数字。

.. code:: js

    DBInsert("mytable", "name,amount", "John Dow", 100)

DBUpdate(tblname string, id int, params string, val...)
-------------------------------------------------------------------------------------------------
该函数通过指定的 **id** 将表中的列值更改为记录。如果此标识符的记录不存在，操作将导致错误。

* *tblname*  – 数据库表名称；
* *id* - 需要修改的数据的ID；
* *params* - 要修改的列的逗号分隔名称列表；
* *val* - ``params`` 中列出的指定列的值列表，可以是一个字符串或数字。

.. code:: js

    DBUpdate("mytable", myid, "name,amount", "John Dow", 100)

DBUpdateExt(tblname string, column string, value (int|string), params string, val ...)
---------------------------------------------------------------------------------------------------------------------------------
函数更新数据中列具有指定值的列。该表应该具有指定列的索引。

* *tblname*  – 数据库表名称；
* *column*  - 列名称；
* *value* - 搜索值；
* *params* - 将列出以逗号分隔的列名称列表，其中 ``val`` 中指定的值将被写入；
* *val* - 在 ``params`` 列出的列中记录的列表值，可以是一个字符串或数字。

.. code:: js

    DBUpdateExt("mytable", "address", addr, "name,amount", "John Dow", 100)
    
数组操作
========

Append(src array, val someType) array
----------------------------------------
将 *val* 追加到 *src* 数组的末尾。若它有足够的容量，其目标就会重新切片以容纳新的元素。否则，就会返回一个新的基本数组。
* *src* - 原数组；
* *val* - 添加到数组中的值。

.. code:: js

    var list array
    list = Append(list, "new_val")

Join(in array, sep string) string
---------------------------------
此函数将 ``in`` 数组的元素合并到指定的 ``sep`` 分隔符的字符串中。

* *in* - ``array`` 类型数组的名称，需要合并的元素；
* *sep* - 一个分隔符字符串。

.. code:: js

    var val string, myarr array
    myarr[0] = "first"
    myarr[1] = 10
    val = Join(myarr, ",")

Split(in string, sep string) array
-------------------------------------------------------------------------------------------------
此函数将 ``in`` 字符串拆分为使用 ``sep`` 作为分隔符的元素，并将它们放入数组中。

* *in* 需要放入数组中的字符串；
* *sep* 分隔符字符串。

.. code:: js

    var myarr array
    myarr = Split("first,second,third", ",")

Len(val array) int
---------------------------------
这个函数返回指定数组中元素的个数。

* *val* - *array* 类型的数组。

.. code:: js

    if Len(mylist) == 0 {
      ...
    }

Row(list array) map
---------------------------------
该函数返回 ``list`` 数组中的第一个 ``map`` 关联数组。如果 ``list`` 为空，则结果将是一个空的 ``map`` 。这个函数主要和 ``DBFind`` 函数一起使用。在这种情况下，不需要指定 ``list`` 参数。

* *list* - ``DBFind`` 函数返回的映射数组。

.. code:: js

   var ret map
   ret = DBFind("contracts").Columns("id,value").WhereId(10).Row()
   Println(ret)

One(list array, column string) string
-------------------------------------------------------------------------------------------------
该函数从 ``list`` 数组的第一个关联数组中返回 ``column`` 键的值。如果 ``list`` 列表为空，则返回 *nil* 。这个函数主要和 ``DBFind`` 函数一起使用。在这种情况下，不需要指定 ``list`` 参数。

* *list* - 由 ``DBFind`` 函数返回的映射数组；
* *column* - 返回密钥的名称。

.. code:: js

   var ret string
   ret = DBFind("contracts").Columns("id,value").WhereId(10).One("value")
   if ret != nil {
      Println(ret)
   }

GetMapKeys(val map) array
------------------------------
该函数返回 *val* 中的键。

* *val* - 映射数组。

.. code:: js

    var val map
    var arr array
    val["k1"] = "v1"
    val["k2"] = "v2"
    arr = GetMapKeys(val) // arr = [k1 k2]

SortedKeys(val map) array
------------------------------
该函数返回 *val* 中的键的递增排序数组。

* *val* - 映射数组。

.. code:: js

    var val map
    var arr array
    val["b1"] = "v1"
    val["a3"] = "v3"
    val["d2"] = "v2"
    arr = SortedKeys(val) // arr = [a3 b1 d2]

合约条件的使用
==============================
CallContract(name string, params map)
-------------------------------------------------------------------------------------------------
该函数按名称调用合约。所有在合约中 ``data`` 中指定的参数都应该列在传输数组中。该函数返回分配给合约中 ``$result`` 变量的值。

* *name* - 合约名称；
* *params* - 与合约的输入数据相关联的数组。

.. code:: js

    var par map
    par["Name"] = "My Name"
    CallContract("MyContract", par)

ContractAccess(name string, [name string]) bool
-------------------------------------------------------------------------------------------------
该函数检查执行的合约的名称是否与参数中列出的名称匹配。通常用来控制对表的访问。在编辑表列或表权限部分中插入和新增列字段时，该函数在权限字段中指定。

* *name* – 合约名称。

.. code:: js

    ContractAccess("MyContract")  
    ContractAccess("MyContract","SimpleContract") 
    
ContractConditions(name string, [name string]) bool
-------------------------------------------------------------------------------------------------
该函数从特定名称合约中调用 ``condition`` 部分。对于这样的合约，``data`` 部分必须是空的。如果条件正确执行，则返回 ``true``。如果在执行过程中产生错误，则父合约也将错误结束。该函数通常用于控制对数据库表的合约访问，并且可以在编辑系统表时在 ``Permissions`` 字段中调用。

* *name* – 合约名称。

.. code:: js

    ContractConditions("MainCondition")  

EvalCondition(tablename string, name string, condfield string) 
-------------------------------------------------------------------------------------------------
函数从 ``tablename`` 表中获取 ``name`` 字段的 ``condfield`` 字段的值，该字段等于 ``name`` 参数，并检查字段 ``condfield`` 的条件是否成立。

* *tablename* - 数据库表名称；
* *name* - 由 ``name`` 字段搜索的值；
* *condfield* - 存储要检查的条件的字段的名称。

.. code:: js

    EvalCondition(`menu`, $Name, `condition`)

GetContractById(id int) string
--------------------------------
该函数通过标识符返回合约名称。如果无法找到合约，将返回空字符串。

 * *id* - 在 *合约* 表中的合约标识符

.. code:: js

    var id int
    id = GetContractById(`NewBlock`)  
    
GetContractByName(name string) int
---------------------------------------- 
函数在 *合约* 中返回一个合约标识符。如果该合约不存在，则返回零值。

 * *name* - 在 *合约* 表中的合约标识符。

.. code:: js

    var name string
    name = GetContractByName($IdContract) 

RoleAccess(id int, [id int]) bool
-------------------------------------
该函数检查调用合约的用户角色标识符是否与参数中列出的标识符之一匹配。 用于控制合约对数据表和其他数据的访问。

* *id* - 角色ID.

.. code:: js

    RoleAccess(1)  
    RoleAccess(1, 3) 

ValidateCondition(condition string, ecosystemid int) 
-----------------------------------------------------------------
该函数编译 ``condition`` 参数中指定的条件。如果在编译过程中发生错误，将会产生错误，并且合约成功调用。此功能旨在检查更改条件时的正确性。

* *condition* - 验证的条件；
* *ecosystemid* - 生态系统标识符。

.. code:: js

    ValidateCondition(`ContractAccess("@1MyContract")`, 1)  
    

账户地址操作
==============================
AddressToId(address string) int
---------------------------------
函数以字符串格式返回他帐户的地址的公民的身份证号码。如果指定了错误的地址，则返回 ``0``。

* *address* - 该帐户地址格式为 ``XXXX -...- XXXX`` 或以数字的形式。

.. code:: js

    wallet = AddressToId($Recipient)
    
IdToAddress(id int) string
---------------------------------
根据其ID号返回一个帐户的地址。如果指定了错误的ID，并返回 ``invalid``。

* *id* - 账户地址ID，数值。

.. code:: js

    $address = IdToAddress($id)
    

PubToID(hexkey string) int
---------------------------------
该函数以十六进制编码方式通过公钥返回帐号地址。

* *hexkey* - 十六进制形式的公钥。

.. code:: js

    var wallet int
    wallet = PubToID("fa5e78.....34abd6")


值和变量操作
==============================
Float(val int|string) float
---------------------------------
该函数将整数 ``int`` 或 ``string`` 转换为浮点数。


* *val* - 整数或字符串。

.. code:: js

    val = Float("567.989") + Float(232)

HexToBytes(hexdata string) bytes
---------------------------------
该函数将十六进制编码的字符串转换为 ``bytes`` 值（字节序列）。

* *hexdata* – 一个包含十六进制符号的字符串。

.. code:: js

    var val bytes
    val = HexToBytes("34fe4501a4d80094")
       
Random(min int, max int) int
---------------------------------
该函数返回min和max之间的一个随机数（min <= result < max）。min和max都是正数。

* *min* - 随机数的最小值；
* *max* - 随机数的最大值。

.. code:: js

    i = Random(10,5000)
   
Int(val string) int
---------------------------------
该函数将字符串值转换为整数。

* *val*  – 包含数字的字符串。

.. code:: js

    mystr = "-37763499007332"
    val = Int(mystr)
    

Sha256(val string) string
---------------------------------
该函数返回 ``SHA256`` 指定字符串的散列。

* *val* - 需要被转换成 ``SHA256`` 的字符。

.. code:: js

    var sha string
    sha = Sha256("Test message")

Str(val int|float) string
---------------------------------
该函数将 ``int`` 或 ``float`` 值转换为字符串。

* *val* - 整数或浮点数。

.. code:: js

    myfloat = 5.678
    val = Str(myfloat)

UpdateLang(state int, appID int, name string, trans string, vde bool)
---------------------------------------------------------------------------
该函数更新内存中的多语言资源，用于更改语言资源。

* *state* - 生态系统ID；
* *appID* - 应用程序ID；
* *name* - 多语言资源的名称；
* *trans* - 已翻译的多语言资源资源；
* *vde* - 是否为vde。

.. code:: js

    UpdateLang($state, $appID, $Name, $Trans, false)

JSONEncode(src int|float|string|map|array) string
-------------------------------------------------------
该函数将数字、字符串或数组 *src* 转换为JSON格式的字符串。

* *src* - 要转换为JSON的数据。

.. code:: js

    var mydata map
    mydata["key"] = 1
    var json string
    json = JSONEncode(mydata)

JSONEncodeIndent(src int|float|string|map|array, indent string) string
--------------------------------------------------------------------------
该函数将数字、字符串或数组 *src* 转换为具有指定缩进的JSON格式的字符串。

* *src* - 要转换为JSON的数据；
* *indent* - 用作缩进的字符串。

.. code:: js

    var mydata map
    mydata["key"] = 1
    var json string
    json = JSONEncodeIndent(mydata, "\n")


JSONDecode(src string) int|float|string|map|array
--------------------------------------------------
该函数可以将src的数据转换为json格式、字符串或数组。

* *src* - 要转换为JSON的数据。

.. code:: js

    var mydata map
    mydata = JSONDecode(`{"name": "John Smith", "company": "Smith's company"}`)

字符串操作
==============================
HasPrefix(s string, prefix string) bool
----------------------------------------
如果字符串 ``s`` 的开始部分是 ``prefix``，返回 ``true``。

* *s* - 需要检查的字符串；
* *prefix* - 需要检查的前缀。

.. code:: js

    if HasPrefix($Name, `my`) {
    ...
    }

Contains(s string, substr string) bool
-------------------------------------------------------------------------------------------------
如果字符串 ``s`` 包含子字符串 ``substr`` ，则返回 ``true`` 。

* *s* - 原始字符串；
* *substr* - 被搜索的字符串。

.. code:: js

    if Contains($Name, `my`) {
    ...
    }    

Replace(s string, old string, new string) string
-------------------------------------------------------------------------------------------------
函数在 ``s`` 字符串中将 ``old`` 字符串的所有字符串替换为 ``new`` 字符串并返回结果。

* *s* - 原始字符串；
* *old* - 需要被替换的字符串；
* *new* - 替换后的字符串。

.. code:: js

    s = Replace($Name, `me`, `you`)
    
Size(val string) int
---------------------------------
该函数返回指定字符串的长度

* *val* - 需要计算长度的字符串。

.. code:: js

    var len int
    len = Size($Name) 
 
Sprintf(pattern string, val ...) string
--------------------------------------------------------------------
该函数根据指定的模板和参数形成一个字符串，可以使用 ``%d(number)，%s(string)，%f(float)，%v(任何类型)``。

* *pattern*  - 输出的数据。

.. code:: js

    out = Sprintf("%s=%d", mypar, 6448)

Substr(s string, offset int, length int) string
-----------------------------------------------------------------------
函数返回从指定字符串开始的子字符串，从偏移量 ``offset`` （从0开始计算）和长度 ``length`` 开始。在不正确的偏移量或长度不正确的情况下，返回空列。如果偏移量和 ``length`` 之和大于字符串大小，则子字符串将从偏移量返回到字符串末尾。

* *val* - 字符串,
* *offset* - 偏移开始处,
* *length* - 长度.

.. code:: js

    var s string
    s = Substr($Name, 1, 10)
    
ToLower(val string) string
------------------------------
返回 *val* 转换为小写形式的副本。

* *val* - 传入字符串。

.. code:: js

    val = ToLower(val)    

ToUpper(val string) string
------------------------------
返回 *val* 转换为大写形式的副本。

* *val* - 传入字符串。

.. code:: js

    val = ToUpper(val)    

TrimSpace(val string) string
------------------------------
该函数删除 *val* 的前后空格、行转换和制表符并返回指定的字符串，。

* *val* - 传入字符串。

.. code:: js

    val = TrimSpace(val)    

字节操作
==============================

StringToBytes(src string) bytes
----------------------------------
该函数将 *src* 转换为字节类型。

* *src* - 字符串。

.. code:: js

    var b bytes
    b = StringToBytes("my string")

BytesToString(src bytes) string
------------------------------------
该函数将 *src* 转换为字符串类型。

* *src* - 字节。

.. code:: js

    var s string
    s = BytesToString($Bytes)

系统变量操作
==============================
SysParamString(name string) string
------------------------------------
该函数返回指定系统参数的值。

* *name* - 参数名称。

.. code:: js

    url = SysParamString(`blockchain_url`)

SysParamInt(name string) int
---------------------------------
该函数以数字形式返回指定系统参数的值。

* *name* - 参数名称。

.. code:: js

    maxcol = SysParam(`max_columns`)

DBUpdateSysParam(name, value, conditions string)
-----------------------------------------------------------------
该函数更新系统参数的值和条件。如果不需要更改值或条件，则在相应参数中指定一个空字符串。

* *name* - 参数名称；
* *value* - 参数的新值；
* *conditions* - 改变参数的条件。

.. code:: js

    DBUpdateSysParam(`fuel_rate`, `400000000000`, ``)
    

日期/时间操作
================================================
函数不允许直接查询，更新等。但是，当对示例中的 ``where`` 条件进行描述时，它允许在获取值时使用 ``PostgreSQL`` 的函数。其中包括处理日期和时间的函数。 例如，当你需要比较 ``date_column`` 列和当前时间时。如果 ``date_column`` 是具有时间戳的类型，那么表达式将是下面的 ``date_column > now()``。如果 ``date_column`` 将时间以 ``Unix`` 格式存储为一个数字，则表达式将是 ``to_timestamp(date_column) > now()``。

.. code:: js

    to_timestamp(date_column) > now()
    date_initial < now() - 30 * interval '1 day'
    
当我们有一个 ``Unix`` 格式值时，我们需要把它写在 ``timestamp`` 类型的字段中。在这种情况下，列出字段时，在此列的名称之前，你需要指定 ``timestamp``。

.. code:: js

   DBInsert("mytable", "name,timestamp mytime", "John Dow", 146724678424 )

如果你有一个时间字符串值，并且你需要将其写入类型为 ``timestamp`` 的字段中，则在此情况下，必须在该值之前指定 ``timestamp``。

.. code:: js

   DBInsert("mytable", "name,mytime", "John Dow", "timestamp 2017-05-20 00:00:00" )
   var date string
   date = "2017-05-20 00:00:00"
   DBInsert("mytable", "name,mytime", "John Dow", "timestamp " + date )
   DBInsert("mytable", "name,mytime", "John Dow", "timestamp " + $txtime )

BlockTime()
-------------
该函数返回到SQL格式的生成时间。应该使用该函数代替 **NOW()** 函数。

.. code:: js

    DBInsert(`mytable`, `created_at`, BlockTime())

VDE功能
==============================
以下功能只能在虚拟专用生态系统(VDE)合约中使用。

HTTPRequest(url string, method string, heads map, pars map) string
---------------------------------------------------------------------------------------------------
这个函数发送一个HTTP请求到一个指定的地址。

* *url* - HTTP请求的地址；
* *method* - 请求的方式（Get或Post）；
* *heads* - 请求头（map格式）；
* *pars* - 请求参数。

.. code:: js

	var ret string 
	var pars, heads, json map
	heads["Authorization"] = "Bearer " + $auth_token
	pars["vde"] = "true"
	ret = HTTPRequest("http://localhost:7079/api/v2/content/page/default_page", "POST", heads, pars)
	json = JSONToMap(ret)

HTTPPostJSON(url string, heads map, pars string) string
---------------------------------------------------------------
这个函数类似于 ``HTTPRequest`` 函数，但是它发送一个 ``POST`` 请求并且在字符串中传递参数。


* *url* - HTTP请求的地址；
* *heads* - 请求头（map格式）；
* *pars* - 请求参数。

.. code:: js

	var ret string 
	var heads, json map
	heads["Authorization"] = "Bearer " + $auth_token
	ret = HTTPPostJSON("http://localhost:7079/api/v2/content/page/default_page", heads, `{"vde":"true"}`)
	json = JSONToMap(ret)

************************************************
系统合约
************************************************
系统合约是在安装期间默认创建的，所有这些合约都是在第一个生态系统中创建。如果从其他生态系统调用这些合约，需要指定其全名，例如 ``"@1NewContract``。

系统合约列表
==============================
NewEcosystem
---------------------------------
该合约创建了一个新的生态系统,要获取新创建的生态系统的标识符，请使用 ``result`` 字段，该字段将在 ``txstatus`` 中返回。参数如下：
   
* *Name string "optional"* - 生态系统的名称。此参数可以稍后进行设置。

MoneyTransfer
---------------------------------
该合约将当前生态系统中的活期账户的通证（Token）转入特定账户。参数如下：

* *Recipient string* - 收件人的帐户以任何格式，数字或 ``XXXX -....- XXXX``；
* *Amount    string* - 交易金额；
* *Comment   string "optional"* - 注释。

NewContract
---------------------------------
该合约在当前的生态系统中创建了一个新的合约。参数如下：

* *Value string* - 合约或者合约的文本信息；
* *Conditions string* - 改变合约的条；
* *Wallet string "optional"* - 用户钱包地址；
* *TokenEcosystem int "optional"* - 生态系统的标识符，当合约被激活时，哪种货币将用于交易。

EditContract
---------------------------------
在当前的生态系统中编辑合约。参数如下：
      
* *Id int* - 编辑的合约ID；
* *Value string* - 合约或合约的文本信息；
* *Conditions string* - 合约变更的条件。

ActivateContract
---------------------------------
将合约绑定到当前生态系统中的帐户。合约可以只与创建合约时指定的账户绑定，合约解除后，该账户将支付合约的执行费用。参数如下：
      
* *Id int* - 要激活的合约的ID

DeactivateContract
---------------------------------
从当前生态系统的帐户中解除合约。只有目前绑定合约的帐户才能解除绑定，合约解除后，将由执行合约的用户支付费用。参数如下：
 
* *Id int* - 绑定合约的标识符。

NewParameter
---------------------------------
该合约为当前的生态系统增加了一个新的参数。参数如下：

* *Name string* - 参数名称；
* *Value string* - 参数值；
* *Conditions string* - 修改参数的条件。

EditParameter
---------------------------------
该合约会更改当前生态系统中的现有参数。参数如下：

* *Name string* - 要更改的参数名称；
* *Value string* - 新值；
* *Conditions string* - 参数改变的新条件。

NewMenu
---------------------------------
该合约在当前的生态系统中添加了一个新的菜单。参数如下：

* *Name string* - 菜单名称；
* *Value string* - 菜单文本信息；
* *Title string "optional"* - 菜单标题；
* *Conditions string* - 菜单改变的权限。

EditMenu
---------------------------------
该合约改变了当前生态系统中的现有菜单。参数如下：

* *Id int* - 要改变的菜单的ID；
* *Value string* - 菜单文本信息；
* *Title string "optional"* - 菜单标题；
* *Conditions string* - 菜单改变的权限。

AppendMenu
---------------------------------
该合约将文本信息添加到当前生态系统中的现有菜单。参数如下：

* *Id int* - 菜单标识符；
* *Value string* - 要添加的文本信息。

NewPage
---------------------------------
该合约在当前的生态系统中增加了一个新的页面。参数如下：

* *Name string* - 合约名称；
* *Value string* - 页面文本信息；
* *Menu string* - 菜单的名称，附在这个页面上；
* *Conditions string* - 修改的权限;
* *ValidateCount int <optional>* - 用于检查页面有效性的节点数量，如果未指定参数，则使用生态系统参数*min_page_validate_count*中的值。该值不能小于*min_page_validate_count*和大于*max_page_validate_count*；
* *ValidateMode int <optional>* - 检查页面的数量。0 - 仅在加载时，1 - 在加载和离开页面时。

EditPage
---------------------------------
此合约会更改当前生态系统中的现有页面。参数如下：

* *Id int* - 要更改的页面的ID；
* *Value string* - 页面的新文本信息；
* *Menu string* - 页面上新菜单的名称；
* *Conditions string* - 页面更改的权限；
* *ValidateCount int <optional>* - 用于检查页面有效性的节点数量，如果未指定参数，则使用生态系统参数*min_page_validate_count*中的值。该值不能小于*min_page_validate_count*和大于*max_page_validate_count*；
* *ValidateMode int <optional>* - 检查页面的数量。0 - 仅在加载时，1 - 在加载和离开页面时。

AppendPage
---------------------------------
合约将文本信息添加到当前生态系统中的现有页面。参数如下：

* *Id int* - 要更改的页面的ID；
* *Value string* - 需要添加到页面的文本信息。

NewBlock
---------------------------------
该合约为当前的生态系统添加了一个带有模板的新页面区块。参数如下：

* *Name string* - 区块名称；
* *Value string* - 区块文本信息；
* *Conditions string* - 新增的条件。

EditBlock
---------------------------------
该合约更改当前生态系统中现有的区块。参数如下：

* *Id int* - 改变区块的ID：
* *Value string* - 新区块的文本信息；
* *Conditions string* - 编辑的条件。

NewTable
---------------------------------
该合约在当前的生态系统中添加一个新数据表。参数如下：

* *Name string* - 数据库表名称；
* *Columns string* - JSON格式的数组 ``[{"name":"...", "type":"...","index": "0", "conditions":"..."},...]``；

  * *name* - 列名称；
  * *type* - 类型， ``varchar、bytea、number、datetime、money、text、double、character``；
  * *index* - 非索引字段 ``0``，创建索引 ``1``；
  * *conditions* - 条件改变列中的数据，读访问权限应该以JSON格式指定。例如， ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``。

* *Permissions string* - JSON格式的访问条件， ``{"insert": "...", "new_column": "...", "update": "..."}``。

  * *insert* - 插入数据的权限；
  * *new_column* - 添加列的权限；
  * *update* - 改变权限的权限。

EditTable
---------------------------------
该合约更改对当前生态系统中数据库表的访问权限。参数如下：

* *Name string* - 数据库表名称；
* *Permissions string* - 以JSON格式访问的权限 ``{"insert": "...", "new_column": "...", "update": "..."}``：

  * *insert* - 插入数据的条件；
  * *new_column* - 新增表列的权限；
  * *update* - 编辑的条件。

NewColumn
---------------------------------
该合约在当前生态系统的表中添加一个新列。参数如下：

* *TableName string* - 数据库表名称；
* *Name* - 列名称；
* *Type* - 类型， ``varchar、bytea、number、money、datetime、text、double、character``；
* *Index* - 非索引字段 - "0"，创建索引 - "1"；
* *Permissions* - 条件改变列中的数据，读访问权限应该以 ``JSON`` 格式指定。例如: ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``。

EditColumn
---------------------------------
此合约会更改当前生态系统中更改表格列的权限。参数如下：

* *TableName string* - 数据库表名称；
* *Name* - 列名称；
* *Permissions* - 条件改变列中的数据，读访问权限应该以 ``JSON`` 格式指定。例如: ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``。

NewLang
---------------------------------
该合约增加了当前生态系统中的语言资源。添加语言资源的权限在生态系统配置的 ``changing_language`` 参数中设置。参数如下：

* *Name string* - 拉丁脚本语言资源名称；
* *Trans* - 语言资源为 ``JSON`` 格式的字符串，其中包含两个字符的语言代码作为键和翻译的字符串作为值。例如: ``{"en": "English text", "ru": "Английский текст", "cn": "中文"}`` 。

EditLang
---------------------------------
该合约更新当前生态系统中的语言资源。进行更改的权限在生态系统配置的 ``changing_language`` 参数中设置。参数如下：

* *Name string* - 语言资源名称；
* *Trans* - 语言资源作为 ``JSON`` 格式的字符串，以两字符的语言代码作为键，将字符串转换为值。例如 ``{"en": "English text", "ru": "Английский текст"}``。
 
NewSign
---------------------------------
此合约为当前生态系统中的合约添加了签名确认要求。参数如下：

* *Name string* - 合约的名称，需要额外的签名确认;
* *Value string* - JSON字符串中参数的描述，其中
    
  * *title* - 消息文本;
  * *params* - 显示给用户的参数数组，其中 ``name`` 是字段名称，``text`` 是参数描述。
    
* *Conditions string* - 改变参数的条件。

例如： *Value* ：

``{"title": "Would you like to sign?", "params":[{"name": "Recipient", "text": "Wallet"},{"name": "Amount", "text": "Amount(GAC)"}]}`` 

EditSign
---------------------------------
该合约用当前生态系统中的签名更新合约的参数。参数如下：

 * *Id int* - 更改签名的标识符;
 * *Value string* - 包含新参数的字符串;
 * *Conditions string* - 更改签名参数的新条件。

Import 
---------------------------------
该合约从 ``.sim`` 文件导入数据到生态系统。参数如下：

* *Data string* - 数据以文本格式导入，此数据是从生态系统导出到 ``.sim`` 文件。

NewCron
---------------------------------
该合约增加了一个新的任务在计时器启动 ``cron`` 。仅在 ``VDE`` 系统中可用。参数如下：

* *Cron string* - 字符串，它以 ``cron`` 格式定时器启动合约；
* *Contract string* - 在 ``VDE`` 中启动的合约名称，该合约在其 ``"data"`` 部分没有参数；
* *Limit int* - 可选字段，其中可以指定合约启动的次数（直到合约被执行这个次数）；
* *Till string* - 结束任务时间的可选字符串（此功能尚未实现）；
* *Conditions string* - 修改任务的权限。

EditCron
---------------------------------
该合约改变了 ``cron`` 中任务的配置以供定时器启动。仅在 ``VDE`` 系统中可用。参数如下：

* *Id int* - 任务ID；
* *Cron string* - 定义以 ``cron`` 格式定时器启动合约的字符串， 要禁用任务，此参数应该是空字符串或不存在；
* *Contract string* - 在 ``VDE`` 中启动的合约名称; 合约不应该在其数据部分有参数；
* *Limit int* - 可选字段，其中可以指定合约启动的次数（直到合约被执行的次数）；
* *Till string* - 结束任务时间的可选字符串（此功能尚未实现）；
* *Conditions string* - 修改任务的权限。