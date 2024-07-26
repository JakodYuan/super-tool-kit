# genReg

生成UVM寄存器模型和寄存器RTL代码

## 1 使用

- 前提：已经有编辑好的寄存器表单excel文件，excel的格式可以参考stk.genExcel，或者直接调用`stk genExcel --mode reg xxx.xlsx`生成寄存器表单的模板。
- 命令：`stk genReg xxx.xlsx --common common.json`
- 输出：生成文件在./out目录下
  - reg_models/xxx_reg_model.sv-------> excel有多个表单页签，就会有多个这种文件，为页签中所以寄存器组成的reg_model
  - reg_models/xxx_reg_test.sv--------> 寄存器扫描测试用例，需要修改地址
  - reg_models/xxx_reg_top_model.sv---> 这是所以xxx_reg_model的汇总
  - reg_models/svk_reg_test.sv--------> 寄存器扫描组件
  - reg_rtl/xxx_sys_cfg.v-------------> 生成的REG RTL代码

common.json需要包含如下内容
- author：用户名，用于显示在生成文件的文件头中
- email：邮箱地址，用于显示在生成文件的文件头中
```json
{
  "author": "Jakod",
  "email": "JakodYuan@outlook.com",
}
```

## 2 寄存器模型

特性：
1. 支持uvm_reg_field自定义，比如invert寄存器，使用默认的uvm_reg_field，则无法通过mirror与RTL比对通过。这里的解决方案分为3步：
   1. 自定义一个invert_reg_field类，并把该文件添加到filelist中(在生成的寄存器模型之前)
   ```sv
   class invert_reg_field extends uvm_reg_field;
        `uvm_object_utils(inv_reg_field)

        local satic bit m_invert = define_access("INVERT"); // 定义一种访问类型

        function new(string name = "inv_reg_field");
            super.new(name);
        endfunction

        task post_write(uvm_reg_item rw);
            if(!predict(~rw.value[0]))
                `uvm_error("post_write", "predict is error")
        endtask
   endclass
   ```
   2. 将寄存器域的access列设置为INVERT，genReg例化的时候会例化invert_reg_field来代替，uvm_reg_field
   ![img](https://img2023.cnblogs.com/blog/898240/202407/898240-20240726154252387-1412828120.png)
   > access类型是RW/RO/RC/WC/WO/这些UVM支持的访问类型，会例化uvm_g_field，否则会选择例化access_reg_field代替uvm_reg_field**待开发**
2. 支持backdoor，在寄存器表单的backdoor列填写路径，即可通过后门访问
3. 打包了一个reg_test用例，用户值需要修改地址即可进行寄存器扫描测试。

## 3 RTL代码

生成的RTL代码使用的是APB接口类型

```sv
module DEMO_CFG;
    input                   pvalid;
    input  [ADDR_WIDTH-1:0] paddr;
    input  [DATA_WIDTH-1:0] pwdata;
    input                   psel;
    input                   pwrite;
    input                   penable;
    output [DATA_WIDTH-1:0] prdata;
    output                  perror;
    output                  pready;
         
    output [16 -1:0] pll;
    output [1  -1:0] soft_reset;
    output [1  -1:0] clk_gate;
    
    
    output [16 -1:0] coff0_0;
    output [16 -1:0] coff1_0;
    output [16 -1:0] coff2_0;
    output [16 -1:0] coff3_0;
    ....
        // TEST1
    wire TEST1_sel;
    wire [18-1:0]TEST1_out;
    wire [16-1:0]pll;
    wire [1-1:0]soft_reset;
    wire [1-1:0]clk_gate;
    assign TEST1_sel = paddr == 0x80000058 ? 1'b1:1'b0
    RW_REG U_TEST1_REG#(
        .WIDTH(18)
    )(
        .clk    (clk),
        .rst_n  (rst_n),
        .sel    (TEST1_sel),
        .in     (pwdata),
        .out    (TEST1_out),
    );
    assign pll = TEST1_out[17:2];
    assign soft_reset = TEST1_out[1:1];
    assign clk_gate = TEST1_out[0:0];


    always@(posedge clk)begin
        if(rst_n == 0)
            prdata <= 0;
        else begin
            case(paddr) 
                0x80000000: prdata <= {pll,soft_reset,clk_gate}
                0x80000008: prdata <= {coff0_0,coff1_0,coff2_0,coff3_0}
                0x80000010: prdata <= {coff0_1,coff1_1,coff2_1,coff3_1}
                0x80000018: prdata <= {coff0_2,coff1_2,coff2_2,coff3_2}
                0x80000020: prdata <= {coff0_3,coff1_3,coff2_3,coff3_3}
                0x80000028: prdata <= {coff0_4,coff1_4,coff2_4,coff3_4}
                0x80000030: prdata <= {coff0_5,coff1_5,coff2_5,coff3_5}
                0x80000038: prdata <= {coff0_6,coff1_6,coff2_6,coff3_6}
                0x80000040: prdata <= {coff0_7,coff1_7,coff2_7,coff3_7}
                0x80000048: prdata <= {coff0_8,coff1_8,coff2_8,coff3_8}
                0x80000050: prdata <= {coff0_9,coff1_9,coff2_9,coff3_9} 
                0x80000058: prdata <= {pll,soft_reset,clk_gate}
                default:prdata <= 0;
            endcase
        end

endmodule
```