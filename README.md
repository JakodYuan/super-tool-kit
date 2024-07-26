# stk(super tool kit)

[TOC]

STK是一个工具箱，包含一系列芯片设计和验证的工具，提高工作效率。

## 1 genExcel

生成excel demo文件，支持接口列表，寄存器表单，crg表单
[stk.genExcel](./doc/python%20stk.genExcel.md)

## 2 e2j

将excel文件转换为json文件，支持接口列表、寄存器表单，crg表单
[stk.e2j](./doc/python%20stk.e2j.md)

## 3 genFile

json+jinja2模板的方式生成文件
[stk.genFile](./doc/python%20stk.genFile.md)

## 4 genEnv

根据接口列表生成验证环境
[stk.genEnv](./doc/python%20stk.genEnv.md)

## 5 genReg

根据excel表生成UVM寄存器模型和RTL
[stk.genReg](./doc/python%20stk.genReg.md)

## 6 genCRG

生成CRG模型，CRG RTL代码
[stk.CRGModel](./doc/python%20stk.genCRG.md)

## 7 bsim

build and simulation，需要在调用命令的目录下，读取一个bsim.yaml的配置文件
[stk.bsim](./doc/python%20stk.bsim.md)

## 8 regress

验证回归工具，支持数据库，个性化显示
[stk.regress](./doc/python%20stk.regress.md)

## 9 regrSever

显示回归数据，按照项目，人员等层级显示回归统计结果。
[stk.regrServer](./doc/python%20stk.regrServer.md)

## 10 instRTL

生成verilog和vhdl的例化，用于代码集成和环境搭建
[stk.instRTL](./doc/python%20stk.instRTL.md)

## 11 parseLog

解析编译或者仿真log，用于判断仿真或者回归的是否通过
[stk.parseLog](./doc/python%20stk.parseLog.md)

## 12 run

命令集合工具
[stk.run](./doc/python%20stk.run.md)