# stk(super tool kit)
STK是一个工具箱，包含一系列芯片设计和验证的工具，提高工作效率。

[TOC]
| 序号 | 命令                         | 说明                                                                      |
| ---- | ---------------------------- | ------------------------------------------------------------------------- |
| 1    | [bsim](#1-bsim)              | build and simulation，封装VCS的编译和仿真命令，实现简单命令完成编译和仿真 |
| 2    | [genCRG](#2-gencrg)          | 生成CRG模型， 和CRG RTL                                                   |
| 3    | [genEnv](#3-genenv)          | 根据接口列表生成验证环境                                                  |
| 4    | [genExcel](#4-genexcel)      | 生成excel demo文件，支持接口列表，寄存器表单，crg表单，bus表单            |
| 5    | [genReg](#5-genreg)          | 根据excel表生成UVM寄存器模型，和CRG RTL代码                               |
| 6    | [instRTL](#6-instrtl)        | 生成verilog和vhdl的例化，用于代码集成和环境搭建                           |
| 7    | [parseLog](#7-parselog)      | 解析编译或者仿真log，用于判断仿真或者回归的是否通过                       |
| 8    | [regress](#8-regress)        | 验证回归工具，自动化用例提交和结果判断，支持数据库和个性化窗口显示        |
| 9    | [regrServer](#9-regrserver)  | 读取regress生成的数据库，在浏览器中以各种图表的形式显示回归数据。         |
| 10   | [genDummy](#10-gendummy)     | 生成RTL的dummy文件                                                        |
| 11   | [instConn](#11-instconn)     | soc集成多个子模块，生成顶层文件                                           |
| 12   | [extFList](#12-extflist)     | filelist中使用宏的情况下，根据宏定义提取filelist                          |
| 13   | [yaml扩展](#13-yaml语法扩展) | yaml语法扩展                                                              |


## 安装方法

### 1 linux

1. 从github下载linux版本stk工具
2. 解压stk到相应目录
3. 把路径添加Path中
    - bash: .bashrc中添加`export PATH=$PATH\:/path/to/stk`；执行`source .bashrc`
    - csh: .cshrc中添加`setenv PATH $PATH\:/path/to/stk`; 执行`source .cshrc`

### 2 windows

1. 从github下载windows版本stk工具
2. 解压到相应目录
3. 把路径添加path环境变量中
4. 在cmd中执行`stk`

