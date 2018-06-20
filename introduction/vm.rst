################################################################################
编译器和虚拟机
################################################################################

本节介绍 *packages/script* 目录中的代码，组织和代码操作，这些代码是智语言（G Language）程序语言虚拟机的编译和操作。本章节主要面向开发人员。

合约的操作如下: 合约和函数使用智语言（G Language）语言编写并存储在生态系统的合约表中。当程序启动时， 它从数据库中读取源代码并将其编译成字节码。当添加或更改合约并将其写入区块链时，将更新数据进行编译，并添加或更新相应的虚拟机字节码。字节码没有保存在任何地方，所以当你退出程序并重新开始时，编译会重新发生。虚拟机由合约，函数，类型等组成。所有生态系统的合约表中描述的全部源代码，将严格地编入一个虚拟机中，并且虚拟机的状态在所有节点上都是相同的。当调用一个合约时，虚拟机不会以任何方式改变它的状态。任何合约或函数的调用执行，发生在每个外部调用创建的独立运行的堆栈上。每个生态系统都可以有一个虚拟生态系统，它可以在一个节点内的区块链外的数据表中工作，并不会直接影响区块链或其他虚拟生态系统。因此，托管这样一个虚拟生态系统的节点需要服从其合约，并为其创建虚拟机。

********************************************************************************
虚拟机
********************************************************************************
虚拟机在内存中的组织结构的介绍。

.. code:: 

    type VM struct {
       Block         
       ExtCost func(string) int64
       FuncCallsDB map[string]struct{}
       Extern bool 
    }
    
* **Block** - 是包含所有信息的最重要的结构；
* **ExtCost** - 是一个函数，返回的是外部调用golang函数的费用；
* **FuncCallsDB** – golang函数的名称集合，调用的费用返回给第一个参数。该函数使用 *EXPLAIN* 来计算数据库费用；
* **Extern** – 创建一个虚拟机时，设置为 ``true``，并在编译代码时不需要调用合约。意思是，在未来可以设置合约是否需要调用。

虚拟机是树状 **Block** 类型。实际上，区块是一个包含一些字节码的独立单元。简而言之，一切放在括号中的语言就是一个区块。例如：

.. code:: 

    func my() {
         if true {
              while false {
                   ...
               }
         }
    } 
    
创建具有一个函数的区块，其中包含一个带if的区块，而该区块又包含一个带while的区块。
    
.. code:: 

    type Block struct {
        Objects map[string]*ObjInfo
        Type int
        Owner *OwnerInfo
        Info interface{}
        Parent *Block
        Vars []reflect.Type
        Code ByteCodes
        Children Blocks
    }
    
* **Objects** –  **ObjInfo** 类型的内部对象模块的集合。例如，如果区块中有一个变量，那么可以通过名称快速获得关于它的信息；
* **Type** – 区块的类型，函数和合约等于 **ObjFunc** 和 **ObjContract**；
* **Owner** –  **OwnerInfo** 结构的链接，其中包含有关编译合约所有者的信息。它是在编制合约时指定的，或是从 **contracts** 表中加载的；
* **Info** – 包含关于对象模块的信息，其信息取决于区块的类型；
* **Parent** – 父区块标识符；
* **Vars** – 带有当前区块变量类型的数组；
* **Code** – 字节码本身，当控制权被转移到这区块时将被执行。例如，在调用函数或循环体的情况下；
* **Children** – 子区块数组。例如：嵌套函数、循环、条件运算符。

**ObjInfo** 的结构。

.. code:: 

    type ObjInfo struct {
       Type int
       Value interface{}
    }
    
**Type** – 对象类型可以采用以下值:

* **ObjContract** – 合约；
* **ObjFunc** – 函数；
* **ObjExtFunc** – 外部golang函数；
* **ObjVar** – 变量；
* **ObjExtend** –  *$name* 变量。

**Value** – 包含每种类型的合适结构，

.. code:: 

    type ContractInfo struct {
        ID uint32
        Name string
        Owner *OwnerInfo
        Used map[string]bool
        Tx *[]*FieldInfo
        Settings map[string]interface{}
    }
    
* **ID** – 合约标识符。调用合约时，该值在区块链中显示；
* **Name** – 合约名称；
* **Owner** – 有关合约的附加信息；
* **Used** – 内部调用的合约名称的集合；
* **Tx** – 合约数据部分中描述的数据数组。

.. code:: 

    type FieldInfo struct {
           Name string
          Type reflect.Type
          Tags string
    }
    
其中 

* **Name** - 字段的名称；
* **Type** - 类型；
* **Tags** – 字段的附加标签；
* **Settings** – 合约设置部分中描述的值的集合。

正如你看到的，这些信息大部分都是与区块结构重复的，被认为是一个构架的缺陷，这是我们想要避免的。

**ObjFunc** 类型的 **Value** 字段包含 **FuncInfo** 结构。

.. code:: 

    type FuncInfo struct {
         Params []reflect.Type
         Results []reflect.Type
        Names *map[string]FuncName
        Variadic bool
        ID uint32
    }
    
* **Params** – 参数类型的数组；
* **Results** – 返回类型的数组；
* **Names** – 尾部函数数据的集合。例如， ``DBFind().Columns ()``。

.. code:: 

    type FuncName struct {
       Params []reflect.Type
       Offset []int
       Variadic bool
    }
    
* **Params** – 参数类型的数组；
* **Offset** – 变量的偏移量数组。实际上，所有使用点函数表示的参数都是可以分配初始值的变量；
* **Variadic** – 如果函数可以将参数的个数作为变量，则为 ``true`` ，
* **ID** – 函数标识符。

**ObjExtFunc** 类型的 **Value** 字段包含 **ExtFuncInfo** 的结构。它描述了golang的功能。

.. code:: 

    type ExtFuncInfo struct {
       Name string
       Params []reflect.Type
       Results []reflect.Type
       Auto []string
       Variadic bool
       Func interface{}
    }
    
匹配参数与 **FuncInfo** 结构相同。

* **Auto** – 传递给golang函数的额外变量数组（如果有的话）。例如， *SmartContract* 类型的sc变量；
* **Func** – golang函数。

**ObjVar** 类型的 **Value** 字段包含 **VarInfo** 结构。

.. code:: 

    type VarInfo struct {
       Obj *ObjInfo
       Owner *Block
    }

* **ObjInfo** – 有关该类型和变量的信息；
* **Owner** – 区块所有者指示符。

**ObjExtend** 对象的 **Value** 字段包含一个带有变量或函数名称的字符串。

虚拟机命令
============================

*cmds_list.go* 文件中描述了虚拟机命令的标识符。字节码是 **ByteCode** 类型结构序列。

.. code:: 

    type ByteCode struct {
       Cmd uint16
       Value interface{}
    }

**Cmd** 字段存储命令标识符， **Value** 字段包含支持值。通常，命令执行堆栈的最后一个元素操作，并在必要时写入结果。

* **cmdPush** – 将 *Value* 字段中的值放入堆栈。例如，它用于把数字和行符放入堆栈；
* **cmdVar** – 将变量值放入堆栈。 *Value* 包含 *VarInfo* 结构的指示符和有关变量的信息；
* **cmdExtend** – 将外部变量值放入堆栈，以 **$** 开头。 *Value* 包含一个带有变量名的字符串；
* **cmdCallExtend** – 调用外部函数，名称以 **$** 开头。函数的参数将从堆栈中取出，并且函数的结果将被放置到堆栈中。值包含函数的名称；
* **cmdPushStr** – 将字符串从 *Value* 放置到堆栈；
* **cmdCall** – 调用虚拟机功能。 *Value* 包含 **ObjInfo** 结构。 该命令适用于 *ObjExtFunc* golang函数和 *ObjFunc* 智语言（G Language）函数。当一个函数被调用时，传输的参数将从堆栈中取出，结果值将返回到堆栈；
* **cmdCallVari** – 类似于 **cmdCall** 命令调用虚拟机功能，但是此命令用于调用具有可变数量参数的函数；
* **cmdReturn** – 用于退出该功能。返回的值被放置到堆栈中。 *Value* 不被使用；
* **cmdIf** – 将控制权转交给 **Block** 结构中的字节码，该结构是传送到 *Value* 字段的指示符。只有在边界堆栈元素返回 *true* 的情况下调用 *valueToBool* 函数才能传递控制权。否则，控制权转移到下一个命令；
* **cmdElse** – 该命令以与 **cmdIf** 命令相同的方式工作，但只有带边界堆栈元素的 *valueToBool* 返回 *false* 时，控制权才会转移到指定的块；
* **cmdAssignVar** – 从 *Value* 中获取 **VarInfo** 变量列表，该值通过 **cmdAssign** 命令获取；
* **cmdAssign** – 将通过 **cmdAssignVar** 命令获取的变量赋值给堆栈；
* **cmdLabel** – 定义一个标签，在 *while* 循环期间控件将被返回；
* **cmdContinue** – 该命令将控制权传递给 **cmdLabel** 标签。执行循环的新迭代。 *Value* 不被使用；
* **cmdWhile** – 检查与所述堆的极端元素 *valueToBool* 并调用 **Block** 传递到值字段，如果该值为 *true*；
* **cmdBreak** – 退出循环；
* **cmdIndex** – 在索引值上获得堆栈的 *集合* 或 *数组*， *Value* 不被使用。 *(map | array) (index value) => (map | array [index value])*；
* **cmdSetIndex** – 将堆栈的边界值分配给 *集合* 或 *数组* 。 *Value* 不被使用。 *(map | array) (index value) (value) => (map | array)*；
* **cmdFuncName** – 添加参数，使用由点 *func name Func (...) .Name (...)* 分开的顺序来传送；
* **cmdError** – 创建一个命令，用于在 *错误* ， *警告* 或 *信息* 中指定错误时终止合约或功能。

以下是直接使用堆栈的命令。 *Value* 字段不被使用。应注意，现在没有实现完全自动类型转换。例如， *string + float | int | decimal => float | int | decimal, float + int | str => float*，但是 *int + string => runtime error*。

* **cmdNot** – 逻辑否定。 *(val) => (! ValueToBool (val))*；
* **cmdSign** – 标识的变化。 *(val) => (-val)*，
* **cmdAdd** – 添加。 *(val1) (val2) => (val1 + val2)*；
* **cmdSub** – 减法。 *(val1) (val2) => (val1-val2)*；
* **cmdMul** – 乘法。 *(val1) (val2) => (val1 * val2)*；
* **cmdDiv** – 除法。 *(val1) (val2) => (val1 / val2)*，
* **cmdAnd** – 逻辑与。 *(val1) (val2) => (valueToBool (val1) && valueToBool (val2))*；
* **cmdOr** – 逻辑或。 *(val1) (val2) => (valueToBool (val1) || valueToBool (val2))*；
* **cmdEqual** – 比较等式，返回bool。 *(val1) (val2) => (val1 == val2)*；
* **cmdNotEq** – 比较不等式，返回bool。 *(val1) (val2) => (val1! = val2)*；
* **cmdLess** – 比较小于式，返回bool。 *(val1) (val2) => (val1 <val2)*；
* **cmdNotLess** – 比较大于等于式，返回bool。 *(val1) (val2) => (val1> = val2)*；
* **cmdGreat** – 比较大于式，返回bool。 *(val1) (val2) => (val1> val2)*；
* **cmdNotGreat** – 比较小于等于式，返回bool。 *(val1) (val2) => (val1 <= val2)*。

如前所述，字节码的执行不会影响虚拟机。例如，它允许在单个虚拟机中同时运行各种功能和合约。 **Runtime** 结构是为了启动函数和合约，以及使用任何表达式和字节代码。

.. code:: 

    type RunTime struct {
       stack []interface{}
       blocks []*blockStack
       vars []interface{}
       extend *map[string]interface{}
       vm *VM
       cost int64
       err error
    }
    
* **stack** – 执行字节码的堆栈；
* **blocks** – 区块调用堆栈。

.. code:: 

    type blockStack struct {
         Block *Block
         Offset int
    }
    
* **Block** –  正在执行的区块指示符；
* **Offset** – 在指定区块的字节码中执行的最后一个命令的偏移量；
* **vars** – 变量堆栈。调用块中的字节码时，其变量将添加到此变量堆栈中。

退出区块后，变量堆栈的大小将返回到先前的值。

* **extend** – 具有外部变量（$ name）值的集合指示符；
* **vm** – 虚拟机指示器；
* **cost** – 执行结果的费用；
* **err** – 出现错误指令。

在RunCode函数中运行字节码。它包含一个循环，为每个字节码命令执行合适的操作。在开始字节码处理前，必须初始化必要的数据。在这里，添加区块。

.. code:: 

    rt.blocks = append(rt.blocks, &blockStack{block, len(rt.vars)})
        
接下来，在堆栈的最后一个元素中得到关于 *tail* 函数的参数信息。
    
.. code:: 

    var namemap map[string][]interface{}
    if block.Type == ObjFunc && block.Info.(*FuncInfo).Names != nil {
        if rt.stack[len(rt.stack)-1] != nil {
            namemap = rt.stack[len(rt.stack)-1].(map[string][]interface{})
        }
        rt.stack = rt.stack[:len(rt.stack)-1]
    }
    
接下来，必须用初始值初始化该区块中定义的所有变量。

.. code:: 

   start := len(rt.stack)
   varoff := len(rt.vars)
   for vkey, vpar := range block.Vars {
      rt.cost--
      var value interface{}
      
由于函数的变量也是变量，所以我们需要按照函数本身所描述的顺序从堆栈的最后一个元素中取出它们。

.. code:: 

    if block.Type == ObjFunc && vkey < len(block.Info.(*FuncInfo).Params) {
        value = rt.stack[start-len(block.Info.(*FuncInfo).Params)+vkey]
    } else {

这里我们用初始值初始化局部变量。

.. code:: 

        value = reflect.New(vpar).Elem().Interface()
        if vpar == reflect.TypeOf(map[string]interface{}{}) {
           value = make(map[string]interface{})
        } else if vpar == reflect.TypeOf([]interface{}{}) {
           value = make([]interface{}, 0, len(rt.vars)+1)
        }
     }
     rt.vars = append(rt.vars, value)
   }
   
接下来, 更新在 *tail* 函数中传输的变量参数的值。

.. code:: 

   if namemap != nil {
     for key, item := range namemap {
       params := (*block.Info.(*FuncInfo).Names)[key]
       for i, value := range item {
          if params.Variadic && i >= len(params.Params)-1 {
          
如果可以传递可变数量的参数，那么将它们组合成一个变量数组。

.. code:: 

                 off := varoff + params.Offset[len(params.Params)-1]
                 rt.vars[off] = append(rt.vars[off].([]interface{}), value)
             } else {
                 rt.vars[varoff+params.Offset[i]] = value
           }
        }
      }
   }
   
之后，就是移除堆栈，从堆栈顶部删除函数参数传输值。现已经将它们的值复制到一个变量数组中。

.. code:: 

    if block.Type == ObjFunc {
         start -= len(block.Info.(*FuncInfo).Params)
    }
    
字节码指令执行循环结束后，必须正确清除堆栈。

.. code:: 

    last := rt.blocks[len(rt.blocks)-1]
    
从堆栈块中移除当前区块。

.. code:: 

    rt.blocks = rt.blocks[:len(rt.blocks)-1]
    if status == statusReturn {

如果从执行的函数成功退出，把返回值添加到堆栈的前一个末尾。

.. code:: 

   if last.Block.Type == ObjFunc {
    for count := len(last.Block.Info.(*FuncInfo).Results); count > 0; count-- {
            rt.stack[start] = rt.stack[len(rt.stack)-count]
            start++
        }
        status = statusNormal
    } else {
   
正如你所看到的，如果这不是我们执行的函数，我们不会还原堆栈状态，但是我们会退出原函数，因为函数中已经执行循环，条件结构也是字节码块。

.. code:: 

        return
      }
    }
    rt.stack = rt.stack[:start]
    
让我们考虑使用虚拟机的其他函数。任何虚拟机都是使用 *NewVM* 函数创建。 **ExecContract**， **CallContract** 和 **Settings** 三个功能被立即添加到每个虚拟机。使用 **Extend** 功能进行添加。

.. code:: 

   for key, item := range ext.Objects {
       fobj := reflect.ValueOf(item).Type()

我们忽略所有转移的对象模块，只看函数。
       
.. code:: 

   switch fobj.Kind() {
   case reflect.Func:
   
根据收到的关于该函数的信息，将 **ExtFuncInfo** 结构的名称添加到顶层集合对象中，

.. code:: 

  data := ExtFuncInfo{key, make([]reflect.Type, fobj.NumIn()), make([]reflect.Type, fobj.NumOut()), 
     make([]string, fobj.NumIn()), fobj.IsVariadic(), item}
  for i := 0; i < fobj.NumIn(); i++ {
  
我们有 **Auto** 参数。通常这是第一个参数，例如 *sc SmartContract* 或 *rt Runtime*。我们不能将它们从智语言（G Language）中转移出来，但是在执行一些golang函数时需要用上。因此，我们指定在调用函数时，将自动使用哪些变量。在这种情况下， **ExecContract**， **CallContract** 函数具有这样的 *Runtime* 参数。

.. code:: 

    if isauto, ok := ext.AutoPars[fobj.In(i).String()]; ok {
        data.Auto[i] = isauto
    }

填写有关参数的信息，

.. code:: 

    data.Params[i] = fobj.In(i)
  }
  
以及返回值的类型，

.. code:: 

   for i := 0; i < fobj.NumOut(); i++ {
      data.Results[i] = fobj.Out(i)
   }
   
向root对象添加一个函数，允许编译器在从合约中使用时找到它们。

.. code:: 

             vm.Objects[key] = &ObjInfo{ObjExtFunc, data}
        }
    }
    
************************************************************
编译
************************************************************    
   
编译函数在 *compile.go* 文件中，从语法分析器中获取通证（Token）数组。编译可以有条件地分为两个层次。在顶层，我们处理函数、合约、代码块、条件语句和循环语句、变量定义等。在较低层，我们编译循环和条件语句中的代码块，或条件表达式。在开始时，让我们考虑一个简单的较低层次情况。将表达式在 **compileEval** 函数转码成字节码。由于我们有一个使用堆栈的虚拟机，因此需要将普通的表达式的中缀记录转换为后缀表示法，或使用逆波兰式。例如，1+2应该被转换为12+，然后把1和2放到堆栈中，然后我们对堆栈中的最后两个元素用加法操作，并把结果写入堆栈。翻译算法本身可以在网上找到。例如， https://master.virmandy.net/perevod-iz-infiksnoy-notatsii-v-postfiksnuyu-obratnaya-polskaya-zapis/。全局变量 *opers = map [uint32] operPrior* 包含翻译成逆波兰式时所需的优先操作。以下变量在函数的开头时定义:

* **buffer** – 字节码命令的临时缓冲区；
* **bytecode** – 字节码指令的最终缓冲区；
* **parcount** – 在调用函数时计算参数的临时缓冲区；
* **setIndex** – 当我们分配给 *map* 或者 *array* 元素时，工作过程中的变量被设置为 *true*。 例如， *a["my"] = 10*。 这种情况下，需要使用特殊的 **cmdSetIndex** 命令。

接下来，从一个循环中，得到一个通证（Token）并进行正确地处理。例如，在找到大括号后

.. code:: 

    case isRCurly, isLCurly:
         i--
        break main
    case lexNewLine:
          if i > 0 && ((*lexems)[i-1].Type == isComma || (*lexems)[i-1].Type == lexOper) {
               continue main
          }
         for k := len(buffer) - 1; k >= 0; k-- {
              if buffer[k].Cmd == cmdSys {
                  continue main
             }
         }
        break main
        
然后停止解析表达式，当移动字符串时，我们查看前一个语句是否是一个操作，以及是否在括号内，否则我们退出并解析其表达式。一般来说，考虑到有必要调用函数、合约以及其它索引，在解析时不满足的条件，该算法本身会对应转换逆波兰式算法。例如，一个计算器，考虑一下解析 *leviement* 类型通证（Token）的选项。我们用它的名称寻找一个变量，函数或合约。如果没有找到，并且这不是函数或合约调用，那么我们指出该错误。

.. code:: 

    objInfo, tobj := vm.findObj(lexem.Value.(string), block)
    if objInfo == nil && (!vm.Extern || i > *ind || i >= len(*lexems)-2 || (*lexems)[i+1].Type != isLPar) {
          return fmt.Errorf(`unknown identifier %s`, lexem.Value.(string))
    }
    
有一种情况，合约被稍后调用。在这种情况下，如果找不到同名的函数和变量，我们希望能有一个合约调用。在语言中，合约和函数调用没有差别。但我们需要通过 **ExecContract** 函数调用合约，该函数是我们在编辑字节码中使用的。
    
 .. code:: 

    if objInfo.Type == ObjContract {
        objInfo, tobj = vm.findObj(`ExecContract`, block)
        isContract = true
    }
    
在 *count* 中，我们将记下目前的变量数量，该值也会随着函数参数数量一起写入堆栈。我们只需在随后每次检测参数时在堆栈的最后一个元素中增加一个单位。

.. code:: 

    count := 0
    if (*lexems)[i+2].Type != isRPar {
        count++
    }
    
由于我们有调用合约参数的 *Used* 列表，因此我们需要为合约被调用的情况做出标记，并在没有 *MyContract()* 参数的情况下调用合约，我们必须添加两个空参数调用 **ExecContract**，因为它应该至少得到两个的参数。

.. code:: 

    if isContract {
       name := StateName((*block)[0].Info.(uint32), lexem.Value.(string))
       for j := len(*block) - 1; j >= 0; j-- {
          topblock := (*block)[j]
          if topblock.Type == ObjContract {
                if topblock.Info.(*ContractInfo).Used == nil {
                     topblock.Info.(*ContractInfo).Used = make(map[string]bool)
                }
               topblock.Info.(*ContractInfo).Used[name] = true
           }
        }
        bytecode = append(bytecode, &ByteCode{cmdPush, name})
        if count == 0 {
           count = 2
           bytecode = append(bytecode, &ByteCode{cmdPush, ""})
           bytecode = append(bytecode, &ByteCode{cmdPush, ""})
         }
        count++

    }
    
如果我们看到有方括号，我们添加 **cmdindex** 命令的索引来获得值。

.. code:: 

    if (*lexems)[i+1].Type == isLBrack {
         if objInfo == nil || objInfo.Type != ObjVar {
             return fmt.Errorf(`unknown variable %s`, lexem.Value.(string))
         }
        buffer = append(buffer, &ByteCode{cmdIndex, 0})
    }
    
**compileEval** 函数直接生成区块中表达式的字节码，但是 **CompileBlock** 函数的形成与表达式中的对象树和字节码都无关。编译也是基于有限状态机来工作，就像词法分析一样，但其中有区别。首先，我们不使用符号进行操作，而使用通证（Token），其次，我们会立即描述 *states* 变量的所有状态和转换。它表示一个集合数组，其中包含按通证（Token）类型划分的索引。每个通证（Token）都具有在 *NewState* 中指定的新状态的 *compileState* 结构，并且如果已经清楚的分析了该结构， *Func* 字段中处理程序的函数就被指定。 

以主要状态为例：
  
.. code:: 

    { // stateRoot
       lexNewLine: {stateRoot, 0},
       lexKeyword | (keyContract << 8): {stateContract | statePush, 0},
       lexKeyword | (keyFunc << 8): {stateFunc | statePush, 0},
       lexComment: {stateRoot, 0},
       0: {errUnknownCmd, cfError},
    },
    
如果遇到换行符或注释，则保持相同的状态。如果我们遇到 **contract** 关键词，那么我们将状态更改为 *stateContract* 并开始解析其构造。如果我们遇到 *func* 关键字，那么我们切换到 *stateFunc* 状态。如果收到其他通证（Token），将会调用错误生成函数。假设我们已经遇到了 *func* 关键字，并且我们已经把状态改成了 *stateFunc*。 

.. code:: 

    { // stateFunc
        lexNewLine: {stateFunc, 0},
        lexIdent: {stateFParams, cfNameBlock},
        0: {errMustName, cfError},
    },
    
由于该函数的名称必须遵循 **func** 关键字，所以当更改字符串时，我们保持相同状态，并与其他所有通证（Token）一起生成相应的错误。如果我们得到通证（Token）标识符的函数名，那么我们在 *stateFParams* 状态获取函数的参数。 在这一过程中,我们调用 *fNameBlock* 函数。应注意，*Block* 结构是使用 *statePush* 标识创建，在这里我们将它从缓冲区中取出并填充我们需要的数据。

.. code:: 

    func fNameBlock(buf *[]*Block, state int, lexem *Lexem) error {
        var itype int

        prev := (*buf)[len(*buf)-2]
        fblock := (*buf)[len(*buf)-1]
       name := lexem.Value.(string)
       switch state {
         case stateBlock:
            itype = ObjContract
           name = StateName((*buf)[0].Info.(uint32), name)
           fblock.Info = &ContractInfo{ID: uint32(len(prev.Children) - 1), Name: name,
               Owner: (*buf)[0].Owner}
        default:
           itype = ObjFunc
           fblock.Info = &FuncInfo{}
         }
         fblock.Type = itype
        prev.Objects[name] = &ObjInfo{Type: itype, Value: fblock}
        return nil
    }
    
**fNameBlock** 函数用于合约和功能（包括嵌套在其它的函数和合约）。它使用正确的结构填充 *Info* 字段，并将其自身写入父区块的“集合对象”中。这样做是为了可以通过给定的名称来调用该函数或合约。同样，我们为所有状态和变量创建函数，并在虚拟机上进行一些操作。至于 **CompileBlock** 函数,它只需要查看所有通证（Token），然后根据其中描述的那些状态来切换 *states*。几乎所有附加的处理代码都是附加标识。 
    
* **statePush** –  *Block* 对象被添加到对象树中；
* **statePop** – 当区块结束时，用大括号结尾；
* **stateStay** – 表示当你更改为新状态时，你需要停留在当前通证（Token）；
* **stateToBlock** – 表示转换到 *stateBlock* 状态。当表达式被处理后，需要在花括号内使用while和if进行区块处理；
* **stateToBody** – 表示过渡到 *stateBody*；
* **stateFork** – 保存通证（Token）的位置。当表达式的标识符或名称以 **$** 开头时，就会有一个函数或任务被调用；
* **stateToFork** – 用于获取存储在 *stateFork* 中的通证（Token）。该通证（Token）将被传递给处理函数；
* **stateLabel** –  用于插入 **cmdLabel** 命令。这是构建的时候需要的；
* **stateMustEval** – 在if和while结构的开始处检查条件表达式的可用性。
    
除了 **CompileBlock** 函数，还应该提到 **FlushBlock** 函数。问题是区块树是构建在独立于现有虚拟机上。更确切地说，我们获取关于虚拟机中的函数和合约的信息，将编译后的区块收集到一个单独的树中。否则，如果在编译期间发生错误，我们将不得不将虚拟机的状态返回到之前的状态。因此，我们将编译树分开，但编译成功后必须调用 **FlushContract** 函数。此函数将我们完成的区块树添加到当前虚拟机。达到这样，编译阶段才认为是完整的。
  
*******************************************************************
词法分析
*******************************************************************    

词法分析器处理传入的字符串，并形成一个以下类型的通证（Token）：

* **sys** - 是系统通证（Token），例如:[]，{}，()；
* **oper** – 运算符 + - / * ；
* **number** – 数值；
* **ident** – 标识符；
* **newline** – 换行符；
* **string** – 字符串；
* **comment** – 注释。

该版本中，在 *script/lextable/lextable.go* 的初步帮助下，构建一个转换表（有限状态机）来解析通证（Token），写入 *lex_table.go* 文件。一般来说， 你可以避免该文件的初步生成，并立即在内存启动时（在init（）中）创建一个转换表。该词法分析发生在 *lex.go* 的 *lexParser* 函数里运行。

**lextable/lextable.go** 文件

在这里，定义了我们的语言使用 *alphabet* 表，并且描述根据下一个收到的符号从一个状态变化到另一个状态的有限状态机。

*states* 包含一个状态列表的JSON对象。

除特定符号外，d用于表示在状态中未指明的所有符号。

n是0x0a，s是空格，q是反引号，Q是双引号，r是字符> = 128，a是AZ和az，1是1-9。

状态的名称是键，可能的值在值对象模块中列出，该新的状态转换每个集合，跟着是转换状态名称。如果我们需要返回初始状态，第三个参数是服务标识，它指示当前符号的处理方式。

例如，我们有主状态和传入的字符 ``/``。
``"/": ["Solidus", "", "push next"]``

**push** 让命令记住它在一个单独的堆栈， **next** 表示转到下一个字符，我们将状态更改为 **solidus** 之后，采取 *next* 字符查看 **solidus** 的状态。

如果出现 ``/`` 或 ``*`` ，那么进入注释状态。很显然，每个注释都有不同后续状态，因为他们以不同的符号结尾。

如果我们有以下字符不是 ``/`` 和 ``*`` ，那么把所有放在堆栈（/）中的东西都记录为一个带有操作符类型的通证（Token），清除堆栈并返回到主状态。

该模块将状态树更改为数值数组，写入到 *lex_table.go* 文件。

在第一个循环中

.. code:: 

    for ind, ch := range alphabet {
    i := byte(ind)
    
	
我们创建允许传入符号的 *alphabet* 表。在 *state2int* 中，我们给每个状态有自己序列的标识符。
    
.. code:: 

    state2int := map[string]uint{`main`: 0}
    if err := json.Unmarshal([]byte(states), &data); err == nil {
    for key := range data {
    if key != `main` {
    state2int[key] = uint(len(state2int))
    
当我们查看所有的状态，包括状态中的每个集合以及该集合每个符号，我们写一个三字节的数字 *[新状态id (0 = main)] + [通证（Token）类型 (0-no token)] + [标识]* 。该两个维度的 *table* 数组包含了其分成的状态和33个以相同顺序排列的 *alphabet* 数组中的传入符号。也就是说，将来我们将用下面的方式来处理该表格。

假设处于 *table* 零线的 *main* 状态中。我们取第一个字符，在 *alphabet* 数组中查找它的索引，并从给定索引的列中获取值。然后从获取到的值放在第二个字节，该字节表示收到的通证（Token）类型。如果解析完成了，在第三个字节中，我们会收到一个新状态的索引。所有这些将在 *lex.go** 文件的 *lexParser* 函数中更详细地讨论。

如果你想添加一些新的字符，你需要将它们添加到 *alphabet* 数组,并增加 *AlphaSize* 常量。如果要组合新添加的符号，则应在状态中对其进行描述，类似于现有的选项。在此之后， ``run lextable`` 来更新 *lex_table.go* 文件。

**lex.go** 文件

**lexParser** 函数产生的词法分析，在直接输入字符串的基础上，返回接收的通证（Token）的序列。 如下是通证（Token）的结构.

.. code:: 

    type Lexem struct {
       Type uint32 // Type of the lexem
       Value interface{} // Value of lexem
       Line uint32 // Line of the lexem
       Column uint32 // Position inside the line
    }

* **Type** – 通证（Token）类型。可以是以下值: *lexSys，lexOper，lexNumber，lexIdent，lexString，lexComment，lexKeyword，lexType，lexExtend*；
* **Value** – 通证（Token）值。值的类型取决于通证（Token）的类型，让我们更详细地考虑一下：

    * **lexSys** – 包括括号，逗号等，在这种情况下，*Type = ch << 8 | lexSys* ，请参阅 *isLPar...isRBrack* 常量，Value本身是uint32(ch)；
    * **lexOper** – 值表示 *uint32* 形式的等同字符序列。例如，请参阅 *isNot...isOr* 常量；
    * **lexNumber** – 数字存储为 *int64* 或 *float64*。如果该数值有一个小数点，那么它是float64；
    * **lexIdent** – 标识符存储为字符串；
    * **lexNewLine** – 换行符。也用来计算线和标记的位置，
    * **lexString** – 行被存储为 *字符串*；
    * **lexComment** – 注释也存储为 *字符串*；
    * **lexKeyword** – 键只存储相应的索引 – 来自 *keyContract ... keyTail* 常量。在这种情况下， *Type = KeyID << 8 | lexKeyword*。此外，应该注意， *true，false，nil* 关键会立即转换为 *lexNumber* 类型的通证（Token），并带有合适的 *bool* 和 *intreface {}* 类型；
    * **lexType** – 值包含对应的 *reflect.Type* 类型的值；
    * **lexExtend** – 以美元符号 **$** 开头的标识符。这些变量和函数从外部传递，因此分配给特殊类型的标识。该值包含的名称以字符串的形式在开头，没有美元符号。

* **Line** – 找到通证（Token）的字符串；
* **Column** – 通证（Token）在字符串中的位置。

让我们来详细考虑 **lexParser** 函数。该 *todo* 函数 – 基于当前状态和发送符号，从字母表找到符号索引，得到新的状态，就是通证（Token）标识符。如果有通证（Token）的话，则从转换表的附加标识。解析本身包括为每个下一个字符连续调用该函数并切换到一个新的状态。当我们看到一个通证（Token）被接收，我们就在输出格式中创建相应的通证（Token）并继续解析。应注意，在解析过程中，我们不会在单独的堆栈或数组中累积符号，因为我们只是将偏移量保存在通证（Token）开始的位置。得到通证（Token）后，我们将下一个通证（Token）的偏移量移到当前的解析位置。

剩下的就是查看解析中使用的标识:

* **push** – 该标识意味着我们开始在一个新的标识中积累符号，
* **next** – 必须将字符添加到当前通证（Token）；
* **pop** – 通证（Token）的接收完成。通常，使用该标识，我们有解析该通证（Token）的标识符类型；
* **skip** – 此标识用于从解析中排除字符。例如，字符串中控制斜线是 *\\n*  *\\r* 。它们会被自动转换。

*******************************************************************
智语言（G Language）
*******************************************************************
    
<十进制数字> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9

<十进制数字> ::= <十进制数字> {<十进制数字>}

<字符码> ::= <任意字符>

<实数> ::= [-] <十进制数字>.[<十进制数字>]

<整数> ::= [-] <十进制数字> | <字符码>

<数值> := <整数> | <实数>

<字母> ::= A | B | … | Z | a | b | … | z | 0x80 | 0x81 | … | 0xFF

<空格> ::= 0x20

<标签> ::= 0x09

<行末> := 0x0D 0x0A

<标签> ::= 0x09
<特殊字符> ::= ! | » | $ | “ | ( | ) | * | + | , | - | . | / | < | = | > | [ |  | ] | _ | | | } | {„ | <标签> | <空格> | <行末>

<符号> ::= <十进制数字> | <字母> | <特殊字符>

<名称> ::= (<字母> | _) {<字母> | _ | <十进制数字>}

<函数名称> ::= <名称>

<变量名称> ::= <名称>

<类型名称> ::= <名称>

<字符串符号 > ::= <tab> | <space> | ! | # | … | [ | ] | …

<字符串元素> ::= {<字符串符号> | » | n | r }

<字符串> ::= » { <字符串元素> } » | ` { <字符串元素> } `

<赋值运算符> ::= =

<一元运算符> ::= -

<二元运算符> ::= == | != | > | < | <= | >= | && | || | * | / | + | -

<运算符> ::= <赋值运算符> | <一元运算符> | <二元运算符>

<参数> ::= <表达式> {,<表达式>}

<合约调用> ::= <合约名称> ( [<参数>] )

<函数调用> ::= <合约调用> [{. <名称> ( [<参数>] )}]

<区块内容> ::= <区块命令> {<行末><区块命令>}

<区块> ::= {<区块内容>}

<区块命令> ::= (<区块> | <表达式> | <变量定义> | <if> | <while> | break | continue | return)

<if> ::= if <表达式><区块> [else <区块>]

<while> ::= while <表达式><区块>

关键词： *break、conditions、continue、contract、data、else、error、false、func、if、info、nil、return、settings、true、var、warning、while*。

**类型**

提供了来自golang的相应类型。

* **bool** – 布尔型；
* **bytes** – []byte{}；
* **int** – 64位整数；
* **address** – 64位无符号整数；
* **array** – []interface{}；
* **map** – map[string]interface{}；
* **money** – decimal.Decimal；
* **float** – 64位浮点数；
* **string** – 字符串。
