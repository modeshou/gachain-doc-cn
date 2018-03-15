################################################################################
REST API v2
################################################################################

Molis软件客户端提供的所有功能，包括认证、生态系统数据接收、错误处理、数据库表操作、用户界面以及合约执行（网络事务），都可以通过政务链平台的 ``REST API`` 获得。 因此，通过使用 ``REST API`` ，开发人员可以在不使用Molis软件客户端的情况下，访问平台的任何功能。

命令调用是通过地址 ``/api/v2/command/[param]`` 执行的，其中 ``command`` 是命令的名称， ``param`` 是附加参数（例如，要更改或接收的资源的名称）。 请求参数应该使用 ``Content-Type: x-www-form-urlencoded`` 。 服务器响应将以 ``JSON`` 格式发送。

********************************************************************************
错误处理
********************************************************************************

在成功执行查询的情况下，返回状态为200。如果发生错误，除了错误状态之外，返回JSON对象，具有以下字段：

* **error** - 错误的标识符；
* **msg** - 错误的文本；
* **params** - 错误的附加参数数组，可以放入错误消息中。

响应示例 

.. code:: 

    400 (Bad Request)
    Content-Type: application/json
    {
        "err": "E_INVALIDWALLET"，
        "msg": "Wallet 1111-2222-3333 is not valid"，
        "params": ["1111-2222-3333"]
    }

错误列表

* **E_CONTRACT** - 没有 ``%s`` 合约；
* **E_DBNIL** - 数据库不存在；
* **E_ECOSYSTEM** - ，生态系统 ``%d`` 不存在；
* **E_EMPTYPUBLIC** - 公钥未定义；
* **E_EMPTYSIGN** - 签名未定义；
* **E_HASHWRONG** - 哈希（Hash）不正确；
* **E_HASHNOTFOUND** - 哈希（Hash）尚未找到；
* **E_INSTALLED** - 平台已安装；
* **E_INVALIDWALLET** - 电子钱包 ``%d`` 无效；
* **E_NOTFOUND** - 内容页面或菜单尚未找到；
* **E_NOTINSTALLED** - 未安装政务链。在这种情况下，你需要通过命令 *install* 安装；
* **E_QUERY** - 数据库查询错误；
* **E_RECOVERED** - 如果出现错误，则API自动恢复；
* **E_REFRESHTOKEN** - 刷新通证（Token）无效；
* **E_SERVER** - 服务器错误。返回golang库函数中是否有错误。该 *MSG* 字段包含错误的文字；
* **E_SIGNATURE** - 签名不正确；
* **E_STATELOGIN** -  ``%s`` 不是该生态系统 ``%s`` 的成员；
* **E_TABLENOTFOUND** - 数据库 ``%s`` 没有找到；
* **E_TOKEN** - 通证（Token）无效；
* **E_TOKENEXPIRED** - 通证（Token）已过期 ``%s`` ；
* **E_UNAUTHORIZED** - 未经授权；
* **E_UNDEFINEVAL** - 值 ``%s`` 未定义；
* **E_UNKNOWNUID** - 未知id；
* **E_VDE** - 虚拟专用生态系统 ``%s`` 不存在；
* **E_VDECREATED** - 虚拟专用生态系统已经创建。


**E_RECOVERED** 表示发现需要寻找并修复的错误。如果系统尚未安装，除了 *install* 命令，任何命令都会返回 **E_NOTINSTALLED** 。**E_SERVER** 可能会在任何响应命令中出现。如果输入不正确参数而出现错误，则可以将其更改为相关错误。在其他情况下，该错误报告无效操作或不正确的系统配置，则需要更详细的调查。如果没有执行登录或会话已过期，除 *install* ， *getuid* ， *login* 之外，其他任何命令都返回 **E_UNAUTHORIZED** 。

********************************************************************************
认证
********************************************************************************

 **JWT 通证（Token）** http://www.jwt.io 用于验证。 收到一个JWT通证（Token）后，你需要把它放在每个查询的头部： ``Authorization: Bearer TOKEN_HERE``。 

getuid
==============================
**GET**/ 返回一个唯一的值， 需要使用你的私钥进行签名， 然后使用 **login** 命令将其发送返回服务器。 目前， 创建临时JWT通证（Token），在调用 **login** 时需要将其传递给 **Authorization**。

.. code:: 
    
    GET
    /api/v2/getuid
    
    Response

* *uid* - 签名行，
* *token* - 在登录中传递的临时通证（Token）。目前，临时通证（Token）的生命周期是5秒。

在不需要授权的情况下，返回如下:

* *expire* - 过期时间（秒）；
* *ecosystem* - 生态系统ID；
* *key_id* - 钱包ID；
* *address* - 钱包地址 ``XXXX-XXXX-.....-XXXX`` 。
    
响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "uid": "28726874268427424"，
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........AU3yPRp64SLO4aJqhN-kMoU5HNYTDplQXbVu0Y"
    }
    
错误: *E_SERVER*   

login
==============================
**POST**/ 用户认证。 **getuid** 命令应该首先被调用，获得唯一的值并签名，在头部中传递一个临时JWT标识，它与getuid一起被接收。 在成功的情况下， 接收到的通证（Token）应该包含在 *Authorization* 头部中。

查询

.. code:: 

    POST
    /api/v2/login
    
* *[ecosystem]* - 生态系统ID。 如果没有指定，该命令将与第一个生态系统一起工作；
* *[expire]* - JWT通证（Token）的生命周期，以秒为单位（默认为36000）；
* *[pubkey]* - 公开十六进制密钥，如果区块链已经存储了一个密钥，那么钱包号应该用 *key_id* 参数传递；
* *[key_id]* - 账户ID或者 ``XXXX-...-XXXX`` 格式，在公钥已存储在区块链中的情况下，不能与 *pubkey* 一起使用；
* *signature* - 通过getuid十六进制接收到的uid签名。

响应

* *token* - JWT 通证（Token）；
* *refresh* - JWT 通证（Token）来扩展会话，应该在 **refresh** 命令中发送；
* *ecosystem* - 生态系统ID；
* *key_id* - 帐户ID；
* *address* - 帐户地址 ``XXXX-XXXX-.....-XXXX`` 的格式，
* *notify_key* - 通知的 *key* 值；
* *isnode* - ``true`` 或 ``false`` - 这个用户是这个节点的所有者；
* *isowner* - ``true`` 或 ``false`` -  这个用户是这个生态系统的所有者；
* *vde* - ``true`` 或 ``false`` - 这个生态系统是否有一个虚拟的专用生态系统。

响应示例 

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........AU3yPRp64SLO4aJqhN-kMoU5HNYT8fNGODp0Y"
        "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........iOiI1Nzk3ODE3NjYwNDM2ODA5MzQ2Iiw"        
        "ecosystem":"1"，
        "key_id":"12345"，
        "address": "1234-....-3424"
    }      

错误: *E_SERVER、E_UNKNOWNUID、E_SIGNATURE、E_STATELOGIN、E_EMPTYPUBLIC*

refresh
==============================
**POST**/ 发布新的通证（Token）并扩展用户会话。如果成功完成，则需要在所有查询的 *Authorization* 头部中发送作为响应收到的通证（Token）。

查询

.. code:: 

    POST
    /api/v2/refresh
    
* *[expire]* - JWT通证（Token）的生命周期，以秒为单位（默认为36000）；
* *token* - 通过以前的 **login** 刷新通证（Token）或 **refresh** 调用。

响应

* *token* - JWT 通证（Token）；
* *refresh* - JWT 通证（Token）来扩展会话，应该在 **refresh** 命令中发送。

响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........AU3yPRp64SLO4aJqhN-kMoU5HNYT8fNGODplQXbVu0Y"
        "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........iOiI1Nzk3ODE3NjYwNDM2ODA5MzQ2Iiw"        
    }     
    
错误: *E_SERVER、E_TOKEN、E_REFRESHTOKEN* 

signtest
==============================
**POST**/ 用指定的私钥签署一个字符串。它只能用于API测试，因为通常私钥不应该发送给服务器。私钥可以在服务器启动目录中找到。

.. code:: 
    
    POST
    /api/v2/signtest
    
* *private* - 十六进制私钥；
* *forsign* - 字符串签名。

响应

* *signature* - 十六进制签名；
* *pubkey* - 发送的十六进制私钥的公钥。
    
响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "signature": "0011fa..."，
        "pubkey": "324bd7..."
    }      

错误: *E_SERVER* 

********************************************************************************
服务命令
********************************************************************************

install
==============================
**POST**/ 启动安装。安装成功后，系统将重新启动。

查询

.. code:: 

    POST
    /api/v2/install
    
* *type* - 安装类型: **PRIVATE_NET、TESTNET_NODE、TESTNET_URL**；
* *log_level* - 日志级别: **ERROR、DEBUG**；
* *first_load_blockchain_url* - 获得区块链的地址，在 *type* 的情况下被指定为 **TESTNET_URL**；
* *db_host* - PostgreSQL数据库的主机。例如： *localhost*；
* *db_port* - PostgreSQL数据库的端口。 例如： *5432*；
* *db_name* - PostgreSQL数据库的名称。 例如： *mydb*；
* *db_user* - PostgreSQL数据库的用户名， 例如， *postgres*；
* *db_pass* - PostgreSQL数据库的密码， 例如： *postgres*；
* *generate_first_block* -  *type* 为 *Private-net* 时，可以设置为 ``0`` 或 ``1``；
* *first_block_dir* - 当 *generate_first_block* 为 0 和 *type* 为 *PRIVATE_NET* 时，第一个区块的目录被指定为 *1block*。

响应

* *success* - 在成功完成的情况下为 ``true``。

响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "success": true
    }      
    
错误: *E_SERVER、E_INSTALLED、E_DBNIL* 

********************************************************************************
数据请求函数
********************************************************************************

balance
==============================
**GET**/ 请求当前生态系统中的帐户余额。

查询

.. code:: 
    
    GET
    /api/v2/balance/{key_id}
    
* *key_id* - 帐户ID可以用任何格式指定 - ``int64`` 、 `` uint64`` 、``XXXX-...-XXXX``。钱包将在用户当前登录的生态系统中进行搜索。
    
响应

* *amount* - 最小单位的账户余额 (例如：qGAC)；
* *money* - 账户余额 (例如：GAC)。
    
响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "amount": "123450000000000000000",
        "money": "123.45"
    }      
    
********************************************************************************
生态系统的应用
********************************************************************************

ecosystems
==============================
**GET**/ 返回一些生态系统。

.. code:: 
    
    GET
    /api/v2/ecosystems/

响应

* *number* - 已安装的生态系统的数量。
    
响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "number": 100，
    }      

vde/create
==============================
**POST**/ 创建当前生态系统的虚拟专用生态系统（VDE）。

.. code:: 
    
    POST
    /api/v2/vde/create

响应

* *result* - 如果已创建VDE，则返回 ``true``。
    
响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "result": true，
    }     
    
错误: *E_VDECREATED*

ecosystemparams
==============================
**GET**/ 返回生态系统参数列表。

查询

.. code:: 
    
    GET
    /api/v2/ecosystemparams/[?ecosystem=...&names=...]
    
* *[ecosystem]* - 生态系统标识符，如果未指定，则返回当前生态系统的参数；
* *[names]* - 接收的参数列表，以逗号分隔，例如： ``/api/v2/ecosystemparams/?names=name,currency,logo``；
* *[vde]* - 需要接收VDE参数时指定 ``true``，在另一种情况下，你不需要指定这个参数。


响应

* *list* - 每个元素存储以下参数的数组:

  * *name* - 参数名称；
  * *value* - 参数值；
  * *conditions* - 更改参数的条件。

响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "list": [{ 
            "name": "name"，
            "value": "MyState"，
            "conditions": "true"，
        }， 
        { 
            "name": "currency"，
            "value": "MY"，
            "conditions": "true"，
        }， 
        ]
    }      
    
错误: *E_ECOSYSTEM、E_VDE*

ecosystemparam/{name}
==============================
**GET**/ 返回当前或指定生态系统中有关 **{name}** 参数的信息。

查询

.. code:: 
    
    GET
    /api/v2/ecosystemparam/{name}[?ecosystem=1]
    
* *name* - 请求的参数名称；
* *[ecosystem]* - 可以指定生态系统ID。当前的生态系统的ID将被默认返回；
* *[vde]* - 需要接收VDE参数时指定 ``true`` 。在另一种情况下，你不需要指定这个参数。

响应
    
* *name* - 参数名称；
* *value* - 参数值；
* *conditions* - 更改参数的条件。
    
响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "name": "currency"，
        "value": "MYCUR"，
        "conditions": "true"
    }      
    
错误: *E_ECOSYSTEM、E_VDE*

tables/[?limit=...&offset=...]
==============================
**GET**/ 返回当前生态系统的数据表列表，你可以添加设置偏移量并指定一些请求的表格。

查询

* *[limit]* - 条目数（默认为25）；
* *[offset]* - 条目开始偏移位置（默认为0）；
* *[vde]* - 指定 ``true``，如果需要接收VDE中的表的列表，则另一种情况下不需要指定该参数。

.. code:: 
    
    GET
    /api/v2/tables
    
响应

* *count* - 表中的条目总数；
* *list* - 每个元素存储以下参数的数组:

  * *name* - 数据表名称（无前缀返回）；
  * *count* - 条目总数。

响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "count": "100"
        "list": [{ 
            "name": "accounts"，
            "count": "10"，
        }， 
        { 
            "name": "citizens"，
            "count": "5"，
       }， 
        ]
    }    
    
错误: *E_VDE* 
    
table/{name}
==============================
**GET**/ 返回当前生态系统中请求的表的信息。

下一个字段返回: 

* *name* - 数据表名称； 
* *insert* - 添加条目的权限；
* *new_column* - 添加列的权限；
* *update* - 更改的权限；
* *columns* - 包含字段的列的数组：名称，类型，更改权限（ ``name,type, perm`` ）。

查询

.. code:: 
    
    GET
    /api/v2/table/mytable
     
* *name* - 表名（没有生态系统ID前缀），
* *[vde]* - 指定 ``true``，如果需要接收VDE参数，则另一种情况下不需要指定该参数，

响应

* *name* - 数据表名称（没有生态系统ID前缀）；
* *insert* - 添加条目的权限；
* *new_column* - 添加列的权限；
* *update* - 更改条目的权限；
* *conditions* - 改变表格配置的权限；
* *columns* - 有关列的信息数组:

  * *name* - 列名称；
  * *type* - 列类型。可能的值包括: ``varchar，bytea，number，money，text，double，character``；
  * *perm* - 更改列中的条目的权限。
    
响应示例 

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "name": "mytable"，
        "insert": "ContractConditions(`MainCondition`)"，
        "new_column": "ContractConditions(`MainCondition`)"，
        "update": "ContractConditions(`MainCondition`)"，
        "conditions": "ContractConditions(`MainCondition`)"，
        "columns": [{"name": "mynum"， "type": "number"， "perm":"ContractConditions(`MainCondition`)" }， 
            {"name": "mytext"， "type": "text"， "perm":"ContractConditions(`MainCondition`)" }
        ]
    }      
    
错误: *E_TABLENOTFOUND、E_VDE*  

list/{name}[?limit=...&offset=...&columns=]
====================================================================================================================================================
**GET**/ 返回当前生态系统中指定表的条目列表。可以指定偏移量和请求数据的表项的数量。 

查询

* *name* - 数据表名称；
* *[limit]* - 条目数（默认为25）；
* *[offset]* - 条目开始偏移位置（默认为0）；
* *[columns]* - 请求列的列表，以逗号分隔，如果未指定，则将返回所有列。id列将在所有情况下返回；
* *[vde]* - 如果你需要从VDE表中接收记录，请指定 ``true`` 。在另一种情况下，你不需要指定这个参数。

.. code:: 
    
    GET
    /api/v2/list/mytable?columns=name
    
响应

* *count* - 表中的条目总数；
* *list* - 每个元素存储以下参数的数组：

  * *id* - 条目ID；
  * 请求列的顺序。

响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "count": "10"
        "list": [{ 
            "id": "1"，
            "name": "John"，
        }， 
        { 
            "id": "2"，
            "name": "Mark"，
       }， 
        ]
    }   
    
row/{tablename}/{id}[?columns=]
=========================================================================================
**GET**/ 返回当前生态系统中具有指定标识的表项。可以指定要返回的列。 

查询

* *tablename* - 数据表名称；
* *id* - 条目ID；
* *[columns]* - 请求列的列表，用逗号分隔。如果未指定，则将返回所有列。id列将在所有情况下返回；
* *[vde]* - 如果需要从VDE表中接收记录，则指定 ``true``，否则不需要指定此参数。

.. code:: 
    
    GET
    /api/v2/row/mytable/10?columns=name
    
响应

* *value* - 接收到的列值的数组：

  * *id* - 条目ID；
  * 请求列的顺序。

响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "values": {
        "id": "10"，
        "name": "John"，
        }
    }   
    
systemparams
==============================
**GET**/ 返回系统参数列表。

查询
 
.. code:: 
    
    GET
    /api/v2/systemparams/[?names=...]

* *[names]* - 请求的参数列表，接收的参数列表可以用逗号分隔指定。 例如： ``/api/v2/systemparams/?names=max_columns，max_indexes``。
 
返回 
 
* *list* - 数组，其中的每个元素包含以下参数：

* *name* - 参数名称；
* *value* - 参数值；
* *conditions* - 更改的条件。

响应示例
 
 .. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "list": [{ 
            "name": "max_columns"，
            "value": "100"，
            "conditions": "ContractAccess("@0UpdSysParam")"，
        }， 
        { 
            "name": "max_indexes"，
            "value": "1"，
            "conditions": "ContractAccess("@0UpdSysParam")"，
        }， 
        ]
    }      

history/{name}/{id}
==============================
 **GET**/ 返回当前生态系统中指定表条目的更新日志。 

请求
 
 * *name* - 数据表名称；
 * *id* - 条目id。
 
返回 
 * *list* - 数组，其中的元素包含所请求条目的修改参数。
 
返回示例
  
.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "list": [
            {
                "name": "default_page"，
                "value": "P(class， Default Ecosystem Page)"
            }，
            {
                "menu": "default_menu"
            }
        ]
    }

********************************************************************************
合约函数操作
********************************************************************************

contracts[?limit=...&offset=...]
=========================================================================================
**GET**/ 返回当前生态系统中的合约列表。可以指定偏移量和一些合约请求。

查询

* *[limit]* - 条目数（默认为25）；
* *[offset]* - 条目开始偏移（默认为0）；
* *[vde]* - 如果需要从VDE接收合约列表，请指定 ``true``，否则你无需指定此参数。

.. code:: 
    
    GET
    /api/v2/contracts

响应

* *count* - 表中的条目总数；
* *list* - 每个元素存储以下参数的数组：

  * *id* - 条目ID；
  * *name* - 合约名称；
  * *value* - 合约的初始值；
  * *active* - 如果合约与账户相关，则等于 ``1`` ，否则等于 ``0`` ；
  * *key_id* - 帐户绑定到合约；
  * *address* - 与合约相关的帐户的地址 ``XXXX-...-XXXX``； 
  * *conditions* - 更改的条件；
  * *token_id* - 生态系统id，使用哪种货币来支付合约。

响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "count": "10"
        "list": [{ 
            "id": "1"，
            "name": "MainCondition"，
            "token_id":"1"， 
            "key_id":"2061870654370469385"， 
            "active":"0"，
            "value":"contract MainCondition {
  conditions {
      if(StateVal(`founder_account`)!=$citizen)
      {
          warning `Sorry， you dont have access to this action.`
        }
      }
    }"，
    "address":"0206-1870-6543-7046-9385"，
    "conditions":"ContractConditions(`MainCondition`)"        
     }， 
    ...
      ]
    }   


contract/{name}
==============================
**GET**/ 提供有关智能合约 **name** 的信息。默认情况下，在当前生态系统中搜索智能合约。

响应

* *name* - 智能合约名称；
* *[vde]* -  如果你需要从VDE接收有关合约的信息，则指定 ``true``，否则不需要指定此参数。

.. code:: 
    
    GET
    /api/v2/contract/mycontract
    
响应

* *name* - 具有生态系统ID的智能合约的名称。例如: ``@{idecosystem}name``；
* *active* - 如果合约与账户绑定，则返回 ``true``，否则返回 ``false``；
* *key_id* - 合约所有者的ID；
* *address* - 与合约相关的帐户的地址 ``XXXX-...-XXXX``；
* *tableid* - 合约表中存储合约条目ID；
* *fields* -  包含有关合约的 **数据** 部分中的每个参数的信息的数组，并包含以下字段：

  * *name* - 字段名称；
  * *htmltype* - html类型；
  * *type* - 参数类型；
  * *tags* - 参数标签。
    
响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "fields" : [
            {"name":"amount"， "htmltype":"textinput"， "type":"int64"， "tags": "optional"}，
            {"name":"name"， "htmltype":"textinput"， "type":"string" "tags": ""}
        ]，
        "name": "@1mycontract"，
        "tableid" : 10，
        "active": true
    }      
    
contract/{name}
==============================
**POST**/ 使用指定名称 **{name}** 调用智能合约。在此之前，调用 ``prepare/{name}`` 命令并签名返回的 *forsign* 字段。在执行成功的情况下，返回一个事务散列，在成功的情况下可以用来获得一个区块编号，否则就是一个错误的文本。

查询

* *name* - 要调用的合约的名称，如果合约是从其他生态系统调用的，则应该指定带有生态系统ID的全名 (*@1MainContract*)；
* *[token_ecosystem]* - 生态系统的标识符，用于支付合约的货币，可以指定为不捆绑合约。在这种情况下， *token_ecosystem* 和当前生态系统中的账户和公钥应该是相同的；
* *[max_sum]* - 可以在执行合约时花费的最大金额，可以在调用与账户无关的合约时指定；
* *[payover]* - 对于不与帐户绑定的合约，可以指定额外的紧急支付 - 这是在计算付款时额外添加到fuel_rate；
* *parameters*， 合约要求；
* *signature* - 从prepare中获得的 *forsign* 值的十六进制签名；
* *time* -  从prepare返回时间；
* *pubkey* - 十六进制公钥的合约签名，请注意，如果公钥已经存储在当前生态系统的密钥表中，则不需要传递它；
* *[vde]* - 如果你从VDE参数调用智能合约，则指定 ``true``，否则不需要指定此参数。

.. code:: 
 
    POST
    /api/v2/contract/mycontract
    signature - hex signature
    time – time， returned by prepare

响应

* *hash* - 发送事务的十六进制hash。

响应示例

.. code:: 

    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634....."，
    }
    
    
prepare/{name}
==============================
**POST**/ 发送一个请求来获取一个字符串来签署指定的合约。这里 **name** 是应该返回签名字符串的事务名称。这个字符串将在 *forsign* 参数中返回。另外，返回的是时间参数，需要和签名一起传递。

查询

* *name* - 合约名称，如果合约是从另一个生态系统调用的，则应指定全名 (``@1MainContract``)；
* *[token_ecosystem]* - 生态系统的标识符，用于支付合约的通证（Token），可以指定给与账户无关的合约。在这种情况下，*token_ecosystem* 和当前生态系统中的帐户和公钥应该是相同的；
* *[max_sum]* - 可以在执行合约时花费的最大金额，可以在调用未绑定合约时指定；
* *[payover]* - 对于没有捆绑的合约，可以指定紧急的额外付款 - 这将是在计算付款时额外添加到fuel_rate；
* *[vde]* - 如果你从VDE参数调用智能合约，则指定 ``true``，否则不需要指定此参数。

.. code:: 
    
    POST
    /api/v2/prepare/mycontract

响应

* *forsign* - 签名的字符串；
* *time* - 时间信息，需要与合约一并发送。

响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "time": 423523768，
        "forsign": "......"， 
    }     
    
txstatus/{hash}
==============================
**GET**/ 返回区块编号或带有特定事务散列的错误，如果 *blockid* 和 *errmsg* 的返回值为空，那么事务还没有包含在该区块中。

查询

* *hash* - 选中交易的hash值。

.. code:: 
    
    GET
    /api/v2/txstatus/2353467abcd7436ef47438
     
响应

* *blockid* - 事务处理成功的情况下区块的编号；
* *result* - 事务操作的结果，通过 **$ result** 变量返回；
* *errmsg* - 错误消息，以防交易被拒绝。
    
响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "blockid": "4235237"，
        "result": ""
    }      


content/{menu|page}/{name}
==============================
**POST**/ 返回名称为 **name** 的页面或菜单的JSON代码，这是模板引擎处理的结果。查询可以有其他参数，可以在模板引擎中使用。如果无法找到页面或菜单，则返回 ``404`` 错误。
    
请求

* *menu|page* - *page* 或 *menu* 收到的页面或菜单；
* *name* - 页面的名称或菜单；
* *[lang]* - 可以指定lcid或两个字母的语言代码来处理相应的语言资源,例如： *en,ru,fr,en-US,en-GB*. 如果没有找到*en-US*文件资源 , 默认使用 *en* 而不是 *en-US* ,
* *[vde]* - 如果从VDE的页面或菜单中接收数据，请指定 ``true`` 。否则，不需要指定这个参数。

.. code:: 
    
    POST
    /api/v2/content/page/default
    
响应

* *menu* - 调用 *content/page/...* 时页面的菜单名称；
* *menutree* - 调用 *content/page/...* 时页面的JSON菜单树；
* *title* - 头部菜单为 *content/menu/...*；
* *tree* - 对象的JSON树。

响应示例

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "tree": {"type":"......"， 
              "children": [
                   {...}，
                   {...}
              ]
        }，
    }      

错误: *E_NOTFOUND*

node/{name}
==============================
**POST** 表示 *node* 调用 **{name}** 智能合约。通过 **HTTPRequest** 函数从VDE合约中调用智能合约。在这种情况下，合约不能用一个账户密钥签名，而是用 *node* 的私钥签名。当其他所有参数与发送合约时的参数相似时，被调用的合约应绑定到一个账户， 因为 *node* 的私钥账户没有足够的通证（Token）来执行合约。如果合约是从VDE合约中调用的，那么应该将授权通证（Token） **$ auth_token** 传递给 **HTTPRequest** 。

.. code:: 

	var pars， heads map
	heads["Authorization"] = "Bearer " + $auth_token
	pars["vde"] = "false"
	ret = HTTPRequest("http://localhost:7079/api/v2/node/mycontract"， "POST"， heads， pars)

返回

.. code:: 
 
    POST
    /api/v2/node/mycontract

响应

* *hash* - 发送事务的十六进制hash。

示例

.. code:: 

    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634....."
    }
