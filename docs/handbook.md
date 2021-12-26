# 使用手册

包含部分功能的操作步骤和一些特性。使用前请按照快速上手的安装步骤保证脚本正确安装。

## AutoInst

---

> 自动生成例化模块连接。

![AutoInst](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/autoinst.gif)

### 操作步骤

1. 写标志`/*autoinst*/`

   将要例化的模块写为如下格式：`module_name inst_name(/*autoinst*/);`

   > ⚠️注意
   >
   > - 格式末尾必须要有分号`;`
   > - 注意格式可不在同一行，只要`(/*autoinst*/)`包含在括号内，括号外为`module_name inst_name`即可；支持端口处带`parameter`的写法。

   

   **e.g.**

   ```verilog
   fetch fetch_inst ( /*autoinst*/ );
   ```

   

   ```verilog
   mem     u_mem(
       /*autoinst*/);
   ```

   

   ```verilog
   writeback 
   #(
       // Parameters
       .BMAN			(BMAN),
   )writeback_inst
   (
       .write_data			(write_data[31:0]),
       /*autoinst*/
   );
   ```

2. 自动例化

   - 使用菜单栏点击`AutoInst(0)`或在命令行输入`:call AutoInst(0)`确认，进行`/*autoinst*/`当前模块自动例化，注意例化时光标必须置于`/*autoinst*/`所在行之前的位置（即在当前行或上一行，若在当前行则必须在`/*autoinst*/`所在列之前）；

   - 使用菜单栏点击`AutoInst(1)`或在命令行输入`:call AutoInst(1)`确认，进行`/*autoinst*/`所有模块自动例化；
   - 上述操作也可以使用快捷键完成。

3. 快捷键

   - 默认键盘快捷键为`<S-F3>`，可在脚本`automatic.vim`中如下位置`配置`快捷键

   ```javascript
   if !hasmapto(':call AutoInst(0)<ESC>')
       map <S-F3>      :call AutoInst(0)<ESC>
   endif
   ```

   - 同时，为避免脚本更新导致的快捷键变更，或想使用自定义快捷键，可通过在`.vimrc(or _vimrc)`中配置相关`mapping`实现覆盖配置。（假设配置为`;ati`）

   ```javascript
   map ;ati      :call AutoInst(0)<ESC>
   ```

4. 添加标志

   通过配置脚本参数，可以添加一些`AutoInst`相关的标志：<code>//INST_NEW</code>、<code>//INST_DEL</code>、<code>io_dir</code>、注释<code>//</code>以及宏定义<code>`ifedf</code>
   
   > - 例化时默认自动在尾部添加`io_dir`，即端口类型`input/output/inout`
   > - 例化时默认若有端口更新，自动在该端口尾部添加`//INST_NEW`
   > - 例化时默认若有端口被删除，自动在所有端口例化之后添加`//INST_DEL`
   > - 例化时默认添加`//`类型注释
   > - 例化时默认添加<code>`ifdef</code>类型的宏定义，包括<code>ifdef/elsif/else/endif</code>
   > - 例化时默认不添加例化模块文件所在位置<code>dir</code>，如打开此配置会在例化模块之前一行添加`//Instance`+`dir`以显示例化模块所在的文件夹地址
   
   可在脚本`automatic.vim`中如下位置`配置`相关参数选择添加/不添加以上内容
   
   ```javascript
   "AutoInst 自动例化配置{{{2
   let s:ati_io_dir = get(g:,'ati_io_dir',1)                   "add //input or //output in the end of instance
   let s:ati_inst_new = get(g:,'ati_inst_new',1)               "add //INST_NEW if port has been newly added to the module
   let s:ati_inst_del = get(g:,'ati_inst_del',1)               "add //INST_DEL if port has been deleted from the module
   let s:ati_keep_chg = get(g:,'ati_keep_chg',1)               "keep changed inst io
   let s:ati_incl_cmnt = get(g:,'ati_incl_cmnt',1)             "include comment line of // (/*...*/ will always be ignored)
   let s:ati_incl_ifdef = get(g:,'ati_incl_ifdef',1)           "include ifdef like `ifdef `endif
   let s:ati_95_support = get(g:,'ati_95_support',0)           "Support Verilog-1995
   let s:ati_tail_not_align = get(g:,'ati_tail_not_align',0)   "don't do alignment in tail when autoinst
   let s:ati_add_dir = get(g:,'ati_add_dir',0)                 "add //Instance ...directory...
   "}}}2
   ```
   
   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置（以`io_dir`为例）
   
   ```javascript
   let g:ati_io_dir = 0
   ```
   
   ![ati_mark_demo](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/ati_mark_demo.gif)
   
5. 重刷

   - 自动保留`/*autoinst*/`上方的例化端口，只对其余端口进行自动例化。

   - 同时，如果配置`ati_keep_chg=1`，若端口连线更改，则不进行重刷，只进行端口对齐操作。

   > 例化时默认`ati_keep_chg=1`（`let s:ati_keep_chg = get(g:,'ati_keep_chg',1)`），即若更改过端口连线，则不进行重刷

   

   ![name&reinst](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/name&reinst.gif)

   ![reinst_with_conn_change](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/reinst_with_conn_change.gif)




## AutoPara

---

> 自动生成例化参数连接。
>
> 操作逻辑基本同`AutoInst`一致，分为两种。
>
> 1. `AutoParaValue`，例化`parameter`的`值`，标志为`/*autoinstparam_value*/`。
> 2. `AutoPara`，例化`parameter`，标志为`/*autoinstparam*/`。

![autopara_normal](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/autopara_normal.gif)

![autopara_value](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/autopara_value.gif)

### 操作步骤

1. 写标志`/*autoinstparam*/`或`/*autoinstparam_value*/`

   整体操作与`AutoInst`一致，参考[AutoInst](#autoinst)

2. 自动例化参数

   整体操作与`AutoInst`一致，参考[AutoInst](#autoinst)

3. 快捷键

   - `AutoPara`默认键盘快捷键为`<S-F4>`，可在脚本`automatic.vim`中如下位置`配置`快捷键

   - `AutoParaValue`默认键盘快捷键为`<S-F5>`，可在脚本`automatic.vim`中如下位置`配置`快捷键

     ```javascript
     if !hasmapto(':call AutoPara(0)<ESC>')
         map <S-F4>      :call AutoPara(0)<ESC>
     endif
     if !hasmapto(':call AutoParaValue(0)<ESC>')
         map <S-F5>      :call AutoParaValue(0)<ESC>
     endif
     ```

   - 同时，为避免脚本更新导致的快捷键变更，或想使用自定义快捷键，可通过在`.vimrc(or _vimrc)`中配置相关`mapping`实现覆盖配置。（假设配置为`;atp`）

     ```javascript
     map ;atp      :call AutoPara(0)<ESC>
     ```

4. 添加标志添加标志

   通过配置脚本参数，可以添加一些`AutoInst`相关的标志：`//PARA_NEW` `//PARA_DEL`，注释`//`以及宏定义<code>ifedf</code>,并支持配置使用`端口`参数例化或使用`所有`参数进行例化。在脚本`automatic.vim`中如下位置`配置`相关参数选择不添加`//PARA_NEW` `//PARA_DEL`，并可通过配置`ONLY_PORT`确定使用哪种参数进行例化。

   > 当前添加注释`//`以及宏定义<code>ifedf</code>在`AutoPara`中只支持使用`端口`参数例化，使用`所有`参数进行例化的会添加无用的注释`//`以及宏定义<code>ifedf</code>，请使用者注意。
   >
   > 另外，注释`//`以及宏定义<code>ifedf</code>只针对`AutoPara`，不论如何配置，`AutoParaValue`均不添加注释`//`以及宏定义<code>ifedf</code>

   

   ```javascript
   "AutoPara 自动参数配置{{{2
   let s:atp_only_port = get(g:,'atp_only_port',0)             "add only port parameter definition,ignore parameter = value; definition
   let s:atp_para_new = get(g:,'atp_para_new',1)               "add //PARA_NEW if parameter has been newly added to the module
   let s:atp_para_del = get(g:,'atp_para_del',1)               "add //PARA_DEL if parameter has been deleted from the module
   let s:atp_keep_chg = get(g:,'atp_keep_chg',1)               "keep changed parameter
   let s:atp_incl_cmnt = get(g:,'atp_incl_cmnt',0)             "include comment line of // (/*...*/ will always be ignored)
   let s:atp_incl_ifdef = get(g:,'atp_incl_ifdef',0)           "include ifdef like `ifdef `endif
   let s:atp_tail_not_align = get(g:,'atp_tail_not_align',0)   "don't do alignment in tail when autopara
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置（以`only_port`为例）

   ```javascript
   let g:atp_only_port = 1
   ```

   例化`端口`参数（`ONLY_PORT=1`）的例子：

   ```verilog
   module writeback
   #(
       parameter BMAN=3, 
       parameter AMAN=45   ,
       parameter WIDTH = 16    , 
       parameter TIME_INTERVAL = 4'd11
   )
   ```

   例化`所有`参数（`ONLY_PORT=0`）的例子：

   ```verilog
   module writeback
   #(
       parameter BMAN=3, 
       parameter AMAN=45   ,
       parameter WIDTH = 16    , 
       parameter TIME_INTERVAL = 4'd11
   )
   (
       ......
   );
       parameter CNT0 = 16'h55, CNT1 = 16'h55;
       parameter CNT2 = 256     ;
       parameter CNT3 = 16'h55  ;
   ```


5. 重刷

   与`AutoInst`一致，参考[AutoInst](#autoinst)

6. 连续声明

   支持特殊的`parameter`连续多个的写法，例如`parameter A = 1, B = 5, C = 6`。

## AutoReg

---

> 自动定义寄存器列表。

![autoreg](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/autoreg.gif)

### 操作步骤

1. 写标志`/*autoreg*/`

   在一个空白行内写入：`/*autoreg*/`

   > ⚠️注意
   >
   > 同一行内：
   >
   > - `/*autoreg*/`前端可以存在空格
   > - `/*autoreg*/`后端可以存在任意字符，操作时会忽略

   

   **e.g.**

   ```verilog
   /*autoreg*/
   ```

   ```verilog
   //multiple spaces
               /*autoreg*/ any_character
   ```

2. 自动生成`reg`

   - 使用菜单栏点击`AutoReg()`或在命令行输入`:call AutoReg()`确认，在当前文本`/*autoreg*/`下方自动生成`reg`，例化时光标位置随意，只要保证当前文本含有包含`/*autoreg*/`的行即可；
   - 上述操作也可以使用快捷键完成。

3. 快捷键

   - 默认键盘快捷键为`<S-F6>`，可在脚本`automatic.vim`中如下位置`配置`快捷键

     ```javascript
     if !hasmapto(':call AutoReg()<ESC>')
         map <S-F6>      :call AutoReg()<ESC>
     endif
     ```

   - 同时，为避免脚本更新导致的快捷键变更，或想使用自定义快捷键，可通过在`.vimrc(or _vimrc)`中配置相关`mapping`实现覆盖配置。（假设配置为`;atr`）

     ```javascript
     map ;atr      :call AutoReg()<ESC>
     ```

4. 添加标志

   默认添加`//REG_NEW` `//REG_DEL`。与`AutoInst`以及`AutoPara`一致，参考[AutoInst](#autoinst)。可在脚本`automatic.vim`中如下位置`配置`相关参数选择不添加以上内容

   ```javascript
   "AutoReg 自动寄存器配置{{{2
   let s:atr_reg_new = get(g:,'atr_reg_new',1)                 "add //REG_NEW if register has been newly added to the module
   let s:atr_reg_del = get(g:,'atr_reg_del',1)                 "add //REG_DEL if register has been deleted from the module
   ...
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置（以`reg_new`为例）

   ```javascript
   let g:atr_reg_new = 0
   ```

5. 重刷

   - 自动保留`/*autoreg*/`范围外的`reg`。即，在`//Start of automatic reg`以及`//End of automatic reg`行之外）不重刷，只对其余端口进行自动生成`reg`。

   - **不支持**修改后的`reg`不重刷的功能。

   ![rereg](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/rereg.gif)

   

6. 无法解析

   如果解析出当前变量为`reg`型，但解析其位宽等信息失败时，可在尾部添加标志位，即`//unresolved`，代表解析变量失败。

   可在脚本`automatic.vim`中如下位置`配置`相关参数选择添加以上内容，默认不添加。

   ```javascript
   "AutoReg 自动寄存器配置{{{2
   ...
   let s:atr_unresolved_flag = get(g:,'atr_unresolved_flag',0) "add //unresolved if reg is unresolved
   ...
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置

   ```javascript
   let g:atr_unresolved_flag = 1
   ```

7. 去除`input/output/inout`

   解析为`reg`型变量自动过滤已经在`input/output/inout`端口处声明过的变量（即在端口处声明的变量默认不会自动声明`reg`）。但如果是`verilog-1995`的写法可能存在依旧需要声明的情况，这时可在脚本`automatic.vim`中如下位置`配置`相关参数选择不自动过滤。

   ```javascript
   "AutoReg 自动寄存器配置{{{2
   ...
   let s:atr_remove_io = get(g:,'atr_remove_io',0)             "remove declared io from autoreg
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置

   ```javascript
   let g:atr_remove_io = 1
   ```


## AutoWire

---

> 自动定义线网列表。

![autowire](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/autowire.gif)

### 操作步骤

1. 写标志`/*autowire*/`

   与`AutoReg`一致，参考[AutoReg](#autoreg)

2. 自动生成`wire`

   - 使用菜单栏点击`AutoWire()`或在命令行输入`:call AutoWire()`确认，在当前文本`/*autowire*/`下方自动生成`wire`，例化时光标位置随意，只要保证当前文本含有包含`/*autowire*/`的行即可；
   - 上述操作也可以使用快捷键完成。

3. 快捷键

   - 默认键盘快捷键为`<S-F7>`，可在脚本`automatic.vim`中如下位置`配置`快捷键

     ```javascript
     if !hasmapto(':call AutoWire()<ESC>')
         map <S-F7>      :call AutoWire()<ESC>
     endif
     ```

   - 同时，为避免脚本更新导致的快捷键变更，或想使用自定义快捷键，可通过在`.vimrc(or _vimrc)`中配置相关`mapping`实现覆盖配置。（假设配置为`;atw`）

     ```javascript
     map ;atw      :call AutoWire()<ESC>
     ```


4. 添加标志

   默认添加`//WIRE_NEW` `//WIRE_DEL`。与`AutoInst`以及`AutoPara`一致，参考[AutoInst](#autoinst)。可在脚本`automatic.vim`中如下位置`配置`相关参数选择不添加以上内容

   ```javascript
   "AutoWire 自动线网配置{{{2
   let s:atw_wire_new = get(g:,'atw_wire_new',1)               "add //WIRE_NEW if wire has been newly added to the module
   let s:atw_wire_del = get(g:,'atw_wire_del',1)               "add //WIRE_DEL if wire has been deleted from the module
   ...
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置（以`wire_new`为例）

   ```javascript
   let g:atw_wire_new = 0
   ```

5. 无法解析

   如果解析出当前变量为`wire`型，但解析其位宽等信息失败，解析信号例化模块失败，获取例化模块相关信号失败等情况时，可在尾部添加标志位，即`//unresolved`，代表解析变量失败。

   可在脚本`automatic.vim`中如下位置`配置`相关参数选择添加以上内容，默认不添加。

   ```javascript
   "AutoWire 自动线网配置{{{2
   ...
   let s:atw_unresolved_flag = get(g:,'atw_unresolved_flag',0) "add //unresolved if wire is unresolved
   ...
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置

   ```javascript
   let g:atw_unresolved_flag = 1
   ```

6. 去除`input/output/inout`

   解析为`wire`型变量自动过滤已经在`input/output/inout`端口处声明过的变量（即在端口处声明的变量默认不会自动声明`wire`）。但如果是`verilog-1995`的写法可能存在依旧需要声明的情况，这时可在脚本`automatic.vim`中如下位置`配置`相关参数选择不自动过滤。

   ```javascript
   "AutoReg 自动寄存器配置{{{2
   ...
   let s:atw_remove_io = get(g:,'atw_remove_io',0)             "remove declared io from autoreg
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置

   ```javascript
   let g:atw_remove_io = 1
   ```

7. 单行例化

   `AutoWire`支持单行例化`wire`的检索，举例：

   ```verilog
   module_name u_inst_name ( .test( a ), .test1( b ), .test2( c ));
   ```


## AutoDef

---

> 自动定义所有信号列表
>
> 可参考[AutoReg](#autoreg)与[AutoWire](#autowire)，`AutoDef`功能上等于`AutoReg`+`AutoWire`。

![autodef](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/autodef.gif)

### 操作步骤

1. 写标志为`/*autodef*/`。快捷键配置位置为

   ```javascript
   if !hasmapto(':call AutoDef()<ESC>')
       map <S-F8>      :call AutoDef()<ESC>
   endif
   ```

   同时，为避免脚本更新导致的快捷键变更，或想使用自定义快捷键，可通过在`.vimrc(or _vimrc)`中配置相关`mapping`实现覆盖配置。（假设配置为`;atd`）

   ```javascript
   map ;atd      :call AutoDef()<ESC>
   ```

   其余步骤及注意事项参考[AutoReg](#autoreg)与[AutoWire](#autowire)。

2. 移动变量

   将`/*autodef*/`范围外的所有已经声明的变量（`reg`或者`wire`）移动到`/*autodef*/`自动定义的变量之后。

   可在脚本`automatic.vim`中如下位置`配置`相关参数选择开启，默认关闭。

   ```javascript
   "AutoDef 自动定义配置{{{2
   let s:atd_move = get(g:,'atd_move',0)                       "move declared define(reg/wire) from other parts to places down below autodef
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置

   ```javascript
   let g:atd_move = 1
   ```

   ![atd_move](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/atd_move.gif)


## AutoArg

---

> 自动声明端口

![autoarg](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/autoarg.gif)

0. 打开`95-support`

   `AutoArg`为`verilog-1995`写法，必须打开相关配置，可在脚本`automatic.vim`中如下位置`配置`相关参数

   ```javascript
   "AutoInst 自动例化配置{{{2
   ...
   let s:ati_95_support = get(g:,'ati_95_support',1)           "Support Verilog-1995
   ...
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置

   ```javascript
   let g:ati_95_support = 1
   ```

1. 写标志`/*autoarg*/`

   将要自动声明的端口写为如下格式：`module module_name (/*autoarg*/);`

   > ⚠️注意
   >
   > - 格式末尾必须要有分号`;`
   > - 注意格式可不在同一行，只要`(/*autoarg*/)`包含在括号内即可。

   

   **e.g.**

   ```verilog
   module fetch  ( /*autoarg*/ );
   ```

   

   ```verilog
   module mem (
           /*autoarg*/);
   ```

   

   ```verilog
   module writeback
   (  /*autoarg*/
       clk,rst,
       write_data
   );
   ```

2. 自动声明

   - 使用菜单栏点击`AutoArg()`或在命令行输入`:call AutoArg()`确认，进行`/*autoarg*/`当前模块自动声明，注意例化时光标必须置于`/*autoarg*/`所在行之前的位置（即在当前行或上一行，若在当前行则必须在`/*autoarg*/`所在列之前）；
   - 上述操作也可以使用快捷键完成。

3. 快捷键

   - 默认键盘快捷键为`<S-F2>`，可在脚本`automatic.vim`中如下位置`配置`快捷键

     ```javascript
     if !hasmapto(':call AutoArg()<ESC>')
         map <S-F2>      :call AutoArg()<ESC>
     endif
     ```

   - 同时，为避免脚本更新导致的快捷键变更，或想使用自定义快捷键，可通过在`.vimrc(or _vimrc)`中配置相关`mapping`实现覆盖配置。（假设配置为`;ata`）

     ```javascript
     map ;ata      :call AutoArg()<ESC>
     ```

4. 自动换行

   默认多个端口一行，到最大宽度后自动进行换行。可在脚本`automatic.vim`中如下位置`配置`相关参数选择不自动换行，默认自动换行。

   ```javascript
   "AutoArg 自动声明配置{{{2
   let s:ata_mode = get(g:,'ata_mode',1)                          "mode 0,no wrap; mode 1 wrap around
   ...
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置

   ```javascript
   let g:ata_mode = 0
   ```

   自动换行：

   ![autoarg](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/autoarg.gif)

   不自动换行：

   ![nowrap](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/nowrap.gif)

5. 自动分类

   默认端口按`input/output/inout`自动进行分类。可在脚本`automatic.vim`中如下位置`配置`相关参数选择不进行自动分类，默认自动分类。

   ```javascript
   "AutoArg 自动声明配置{{{2
   ...
   let s:ata_io_clsf = get(g:,'ata_io_clsf',1)                    "input/output/inout classified
   ...
   "}}}2
   ```

   也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置

   ```javascript
   let g:ata_io_clsf = 0
   ```

   ![io_clasf](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/io_clasf.gif)


## RtlTree

---

> 通过Rtl树观察代码结构
>
> 此功能完全参考zhangguo的脚本[automatic for Verilog & RtlTree](https://www.vim.org/scripts/script.php?script_id=4067)，只进行`tags`生成的`vimscript`集成以及文件名跨文件夹的重构。同时，此功能开启后由于自动生成`tags`，因此可以通过`<C-]>`进行模块的快速跳转。

![callout_rtl](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/callout_rtl.gif)

### 操作步骤

1. 打开`Rtl`

   命令行输入`RtlTree`（或直接使用缩略的`Rtl`）确认即可

   ```javascript
   :RtlTree
   ```

2. 跳转

   - 鼠标操作

     单击跳转至例化位置，双击跳转至模块内部

     ![mouse](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/mouse.gif)

   - 键盘操作

     单击`~`，`-`，`+`位置跳转至模块内部，单击同一行其他位置跳转至例化位置

     ![fastkey](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/fastkey.gif)

   

## 跨文件夹

---

> 使用`AutoInst`、`AutoPara`、`AutoParaValue`、`AutoWire`、`AutoDef`、`RtlTree`等功能时，可能例化的模块不在当前文件夹下，而在上一层或下一层文件夹的某个位置，此时需要进行配置

1. 跨文件夹设置

   在代码中添加如下格式的内容`声明`文件夹即可保证代码文件被搜索到：

   ```verilog
   //Local Variables:
   //verilog-library-directories:("." "./aaa/bbb/ccc")
   //verilog-library-directories-recursive:0
   //End:
   ```

   - `verilog-library-directories`为要选择的文件夹，文件夹之间以空格隔开；

   - `verilog-library-directories-recursive`为是否进行文件夹递归搜索，即搜索所有选择文件夹及其子文件夹；

     > 假设需要选择当前文件夹以及其下所有子文件夹作为例化搜索的对象，则配置为：
     >
     > `//verilog-library-directories:(".")`
     > `//verilog-library-directories-recursive:1`
     >
     > ⚠️注意
     >
     > 如果使用递归搜索，请勿采用重叠的文件夹，否则递归会报错；例如上述例子中，`"."` 当前文件夹递归搜索包含`"./aaa/bbb/ccc"`子文件夹，若此时使用递归搜索则重复调用时会报错。

     ![CrossDir](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/cross_dir.gif)

2. 默认位置

   如果不配置跨文件夹的选项，默认会以打开`vim`的位置作为搜索顶层往下递归搜索相关`.v`或`.sv`文件。

   注意不要在`桌面`或者`盘符根目录`等位置打开文件并使用脚本，否则搜索可能会卡死。（暂时不考虑修复为自动切换地址到文件位置，因为与`RtlTree`部分功能冲突）


## 位置对齐

---

> 配置自动生成代码的对齐位置

1. 对齐位置

   可在脚本`automatic.vim`中如下位置`配置`相关参数选择使用自动函数时的对齐位置（下列为`AutoInst`相关配置，其他函数`AutoArg`、`AutoPara`、`AutoReg`、`AutoWire`、`AutoDef`、`AutoArg`配置同理，同时也可通过在`.vimrc(or _vimrc)`中配置相关`global`参数实现配置）

   ```javascript
   "AutoInst {{{3
   "start position
   let s:ati_st_pos = 4
   let s:ati_st_prefix = repeat(' ',s:ati_st_pos)
   "name position
   let s:ati_name_pos_max = 32 
   "symbol position
   let s:ati_sym_pos_max = 64 
   "}}}3
   ```

   `st_pos`：起始位置`start position`

   `name_pos_max`：信号名对齐位置`name max position`

   `sym_pos_max`：第二个括号对齐位置`symbol max position`

   如果信号长度过长（长于设定的对齐位置），则会按照`最长信号长度+4`为新的对齐位置进行对齐。

   ![autopara](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/pos.png)

   

2. 行尾对齐

   可在脚本`automatic.vim`中如下位置`配置`相关参数选择进行行尾对齐/不对齐（默认0，进行行尾对齐）

   ```javascript
   "AutoInst 自动例化配置{{{2
   ......
   let s:ati_tail_not_align = get(g:,'ati_tail_not_align',0)   "don't do alignment in tail when autoinst
   "}}}2
   "AutoPara 自动参数配置{{{2
   ......
   let s:atp_tail_not_align = get(g:,'atp_tail_not_align',0)   "don't do alignment in tail when autopara
   "}}}2
   "AutoReg 自动寄存器配置{{{2
   ......
   let s:atr_tail_not_align = get(g:,'atr_tail_not_align',0)   "don't do alignment in tail when autoreg
   "}}}2
   "AutoWire 自动线网配置{{{2
   ......
   let s:atw_tail_not_align = get(g:,'atw_tail_not_align',0)   "don't do alignment in tail when autowire
   "}}}2
   ```



   - `AutoInst`行尾对齐

     ![image-20210612222934861](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/image-20210612222934861.png)

     

- `AutoInst`行尾不对齐

  ![image-20210612222827132](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/image-20210612222827132.png)

- `AutoPara`的行尾对齐

  ![image-20210612223220307](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/image-20210612223220307.png)

- `AutoPara`的行尾不对齐

  ![image-20210612223427942](https://cdn-1301954091.cos.ap-chengdu.myqcloud.com/blog/vimscript-automatic/image-20210612223427942.png)

     

- `AutoReg`、`AutoWire`、`AutoDef`及`AutoArg`的行尾对齐与`AutoInst`以及`AutoPara`类似