# stk instRTL

生成verilog文件中module的例化，或者vhdl文件中entity的例化。生成例化的格式都是verilog格式的。

## 使用方式

- verilog
```sh
stk instRTL xxx.sv              # 生成的inst会打印在窗口中
stk instRTL xxx.sv -c           # 生成的inst拷贝到了剪切板中，可以直接在使用的地方粘贴即可
stk instRTL xxx.sv -f xx.txt    # 生成的inst写入到文件中
```

- vhdl
```sh
stk instRTL -v xxx.v             # 生成的inst会打印在窗口中
stk instRTL -v xxx.v -c          # 生成的inst拷贝到了剪切板中，可以直接在使用的地方粘贴即可
stk instRTL -v xxx.v -f xx.txt   # 生成的inst写入到文件中
```

## RTL格式说明

无论verilog和还是vhdl，端口格式都很多样，如果考虑所以可能出现的格式，程序编写非常复杂。然而生僻的写法，基本上不会被使用，甚至有些公司规范不允许使用，所以，程序对输入的RTL端口格式不是完全支持，而是支持常用格式。

### verilog

- ANSI写法

```sv
module module_name#(
    parameter name = default_value,             // 支持省略类型，default type is int
    parameter real name = default_value
)(
    input name,                                 // 支持端口省略方向，省略类型
    output logic [7:0] name,
    logic [3:0] name,                          
    name,                                       // 支持类型方向同时省略
)
    parameter name = default_value;             // 支持在module block内添加parameter
endmodule
```


- Non-ANSI写法

```sv
module module_name(name0, name1, name2, name3)
    parameter name = default_value,             // 支持省略类型，default type is int
    parameter real name = default_value
    input name0;                                 // 支持省略类型
    output logic [7:0] name1;
    output logic [3:0] name2,name3;              // 支持同时写多个端口信号            
endmodule
```

- import
module u_xx后面包含`import xx;`语句
```sv
module module_name
    import xxxx::*;
#(
    parameter width = 1
)(
    ports
);
```

- include

port端口通过include文件的方式添加进来
```sv
module module_name(
    `include "rtl_port.sv"
);
```

### vhdl

```sv
library ieee;
use ieee.std_logic_1164.all;

entity my_module is
    generic (
        NUM     : natural := 10;
        WIDTH   : natural := 8  // 参数WIDTH，默认值为8
    );
    port (
        data        : in  std_logic_vector(WIDTH-1 downto 0);
        clk,reset   : in  std_logic;                                // 支持同时写多个信号
        q           : out std_logic_vector(WIDTH-1 downto 0)
    );
end entity my_module;
```

## 不支持场景
