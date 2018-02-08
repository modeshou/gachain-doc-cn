################################################################################
签名合约
################################################################################

由于编写合约的语言允许执行封闭的合约，所以可以在用户不知道已经运行外部合约的情况下完成这样一个封闭的合约，这可能导致用户对其未经授权的交易进行签名，比如，资金来自其帐户。

假设有一个合约 *MoneyTransfer*:

.. code:: js

    contract MoneyTransfer {
        data {
          Recipient int
          Amount    money
        }
        ...
    }

如果用户调用的合约中 ``MoneyTransfer("Recipient,Amount", 12345, 100)`` 被标记，100个通证（Token）将被转移到钱包 *12345* 。在这种情况下，签署外部合约的用户仍然不知道在交易。如果 *MoneyTransfer* 合约在合约的调用中需要额外的用户签名，则可能会排除这种情况。要做到这点：

1. 在 *MoneyTransfer* 合约的 *Data* 部分中添加一个具有 *optional* 和 *hidden* 参数的字段，该字段允许在直接调用合约时不需要附加签名，这样 **Signature** 字段中将有签名。

.. code:: js

    contract MoneyTransfer {
        data {
          Recipient int
          Amount    money
          Signature string "optional hidden"
        }
        ...
    }

2. 在 *Signatures* 表中添加包含以下内容的条目:

* *MoneyTransfer* 合约名称，
* 其值将显示给用户的字段名称及其文本信息，
* 确认后显示的文本信息。

在当前的例子中，它将指定两个字段 **Receipient** and **Amount**:

* **Title**: 你同意向该接收人发送款项吗？
* **Parameter**: 接收人信息: 钱包ID，
* **Parameter**: 金额信息: 金额 (qGAC)。

如果调用合约 *MoneyTransfer(“Recipient, Amount”, 12345, 100)* ，将显示系统错误： ``Signature is not defined`` 。如果合同调用如下: ``MoneyTransfer(“Recipient, Amount, Signature”, 12345, 100, ”xxx...xxxxx”）`` ，系统错误将发生在签名验证。合约调用后，验证信息如下： ``time of the initial transaction, user ID, the value of the fields specified in the signatures table`` ，不可能伪造签名。

为了让用户在调用 *MoneyTransfer* 合约的同时看到汇款确认，必须添加一个具有任意名称和 *String* 类型字段，并使用可选参数 *Signal：Contractname* 。在随后调用 *MoneyTransfer* 合约时，只需转发此参数即可。外部合约的数据部分还必须描述合约的参数（它们可能是隐藏的，但仍会在确认后显示）。例如，

.. code:: js

    contract MyTest {
      data {
          Recipient int "hidden"
          Amount    money
          Signature string "signature:send_money"
      }
      func action {
          MoneyTransfer("Recipient,Amount,Signature",$Recipient,$Amount,$Signature)
      }
    }

当发送 *MyTest* 合约时，需要用户提供转帐到指定钱包的额外确认。如果在附上的合约中列出了 *MoneyTransfer(“Recipient,Amount,Signature”,$Recipient, $Amount+10, $Signature)* 等其他值，将出现无效签名错误。
