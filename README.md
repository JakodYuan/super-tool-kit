# stk(super tool kit)
STK是一个工具箱，包含一系列芯片设计和验证的工具，提高工作效率。


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

## 1 bsim

bsim是stk的编译和仿真工具，通过bsim.yml配置文件实现编译和仿真, 支持内嵌python脚本。

### 1.1 命令

bsim使用分为2步法和3步法：
2步法：
```sh
stk bsim -step2 -b           # compile+simulation,
stk bsim -step2 -b -ns       # compile only,
```
3步法：
```sh
stk bsim -step3 -a -ns       # analysis(all),
stk bsim -step3 -a -b -ns    # analysis(all)+elaboration,
stk bsim -step3 -a           # analysis(all)+elaboration+simulation,
stk bsim -step3 -a -b        # analysis(all)+elaboration+simulation,
stk bsim -step3 -b -ns       # elaboration
stk bsim -step3 -b           # elaboration+simulation,
stk bsim -step3 -i 2         # analysis(only flist2)+elaboration+simulation(recommend),
stk bsim -step3 -i 2 -ns     # analysis(only flist2)
stk bsim -cfg bsim.yml -b  # 编译和仿真
stk bsim -cfg bsim.yml     # 仿真
```

### 1.2 bsim.yml示例

```yaml
bsim_cfg(bsim_base_cfg):
  com:
    flists:
      - vhdl|work|$$env_dir/tb/vh.f
      - vlog|work|$$env_dir/tb/abc.f
    fsdb: true
  sim:
     opts+: >-
        +uvm_set_verbosity=\"*,connect_ph,UVM_HIGH,build\"
```

### 1.3 使用示例

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224143716276-443197121.png)

## 2 genCRG

参考：[https://github.com/JakodYuan/SVK-CRG-TEST](https://github.com/JakodYuan/SVK-CRG-TEST)

## 3 genEnv

genEnv根据接口列表生成验证环境

### 3.1 命令

```sh
stk genEnv --common common.yaml GPU.xlsx # GPU.xlsx是接口列表文件，可以通过stk genExcel -m itf GPU.xlsx生成模板
```

### 3.2 配置文件

需要一个common.yaml设置一些通用配置

```yaml
author: Jakod
email: JakodYuan@outlook.com
prefix: ''
proj: kirin
top: gpu
```

### 3.3 使用示例

生成环境目录结构如下：
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224144002006-2062683460.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224143905793-1856195102.png)



## 4 genExcel

genExcel生成excel模板，支持接口列表，寄存器表单，crg表单，bus表单。

### 4.1 命令

```sh
stk genExcel -m itf|reg|crg|bus xxx.xlsx
```

### 4.2 使用示例

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224144146357-215518178.png)



## 5 genReg

genReg根据excel表生成UVM寄存器模型和RTL代码，还包括软件使用macro文件和覆盖率文件。

### 5.1 命令

```sh
stk genReg -c common.yaml xxx.xlsx 
```

### 5.2 配置文件

需要一个common.yaml设置一些通用配置

```yaml
author: Jakod
email: JakodYuan@outlook.com
prefix: ''
proj: kirin
top: gpu
```

### 5.3 模板文件

genReg是根据模板生成RTL代码，理论上可以生成任意类型的寄存器，内部只带`RW/SRW/RC/WC/RO/W1C/W1P/MAX_REG/MIN_REG/SAT_CNT_RC_REG/SAT_CNT_WC_REG/CYC_CNT_RC_REG/CYC_CNT_WC_REG/W1P`这13种寄存器类型。
内部自带3种接口类型：axi-lite/apb/ahb
支持中断寄存器，支持安全，地址重叠检查，寄存器重名检查，寄存器数组等功能。

这些功能都是模板定义的，用户可以自定义模板，扩展或者覆盖相关功能。

### 5.4 使用示例

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224144516419-1652994296.png)

## 6 instRTL

instRTL根据接口列表生成verilog和vhdl的例化代码，用于代码集成和环境搭建。

### 6.1 命令

```sh
stk instRTL xxx.v
```

### 6.2 使用示例

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224144758115-960273099.png)


## 7 parseLog

parseLog用于解析编译或者仿真log，用于判断仿真或者回归的是否通过。该功能集成到了regress中。

### 7.1 命令

```sh
stk parseLog -m PASSED -c errors -n error,failed,fatal xxx.log
```

-m: 表示必须包含的内容
-c: 可以包含的内容，用户把-n的内容加入白名单
-n: 不可以包含的内容

## 7.2 使用示例

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224144910976-394032177.png)

## 8 regress

regress是stk的回归工具，用于执行regr.yml中定义的回归测试。

### 8.1 命令

```sh
stk regress -cfg regr.yml
```

### 8.2 regr.yml示例

```yaml
regr_cfg:
  proj_name: Pangu
  subsys_name: GPU
  database_en: True
  timeout: 10
  dist_algo: WRR # RR/WRR/SWRR
  max_run_num: 1000

 
build_common:
  must_contain: # 没有must_contain关键字
 
sim_common:
  seed: '{{SEED}}'
  weight: 1
  times: 1
 
# 随机用例，通过配置cfg来实现用例
rand_sim(sim_common):
  work_dir: work
  tc: tc001_random
  opt: ''
  times: 10
  pname: pname
  run_cmd: stk bsim -tc=$tc -seed=$seed -work_dir=$work_dir -p=$$(ENV_ROOT_DIR)/tc/cfg/$pname.cfg -pname=$pname $opt
  fail_cmd: stk bsim -tc=$tc -seed=$seed -work_dir=debug -p=$$(ENV_ROOT_DIR)/tc/cfg/$pname.cfg -pname=$pname $opt -fsdb  # 出错后dump波形，使用的是debug的build进行仿真
  pass_cmd: rm -rf $(work_dir)/log/$(tc)_$(pname)_$(seed).log # pass后删除log
  log: $(work_dir)/log/$(tc)_$(pname)_$(seed).log
 
# 直接用例
test_sim(sim_common):
  work_dir: work
  tc: tc001_sanity
  opt: ''
  times: 10
  run_cmd: stk bsim -tc=$tc -work_dir=$work_dir -seed=$seed $opt
  fail_cmd: stk bsim -tc=$tc -work_dir=debug -seed=$seed $opt -fsdb
  pass_cmd: rm -rf $(work_dir)/log/$(tc)_$(seed).log
  log: $(work_dir)/log/$(tc)_$(seed).log

builds:
  build0(build_common):
    work_dir: work
    run_cmd: stk bsim -work_dir=$work_dir -step2 -b -ns
    log: $work_dir/log/cmp.log
 
  build1(build_common):
    work_dir: debug
    run_cmd: stk bsim -work_dir=$work_dir -step3 -b -ns -fsdb
    log:
      - $work_dir/log/vh1_an.log
      - $work_dir/log/tb2_an.log
      - $work_dir/log/db3_an.log
      - $work_dir/log/elab.log
 
 
simulations:
  tc001_sanity(test_sim):
    tc: tc001_direct_test1
 
  tc002_sanity(test_sim):
    tc: tc002_direct_test2
 
  tc003_sanity(test_sim):
    tc: tc002_direct_test2
    opt: -so '+SVK_MAX_TIME=30000'
 
  random1(rand_sim):
    tc: tc001_sanity
    pname: pname1
    seeds: [132456, 468465, 56565] # 指定仿真的种子，如果times>seeds，后面的会使用随机种子。
    times: 3
 
  random2(rand_sim):
    tc: tc003_sanity
    pname: pname2
    times: 0  # 这条用例不仿真
```

### 8.3 使用示例

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224145639578-775347016.png)

## 9 regrServer

regrServer是stk的回归服务器，用于显示回归结果。

### 9.1 命令

```sh
stk regrServer
```

### 9.2 使用示例

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224145751454-686800493.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224145804669-1935201754.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224155232835-1792852333.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224145852828-1557322534.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224155347141-777489458.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224155445965-1196590207.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224150002129-2093732769.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224155607379-1792047679.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224150035618-1153635653.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224155747316-1824809240.png)

## 10 genDummy

### 10.1 命令

```sh
stk genDummy xxx.v
```

### 10.2 使用示例

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224155828098-507179731.png)

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224155911459-1698976401.png)

## 11 instConn

instConn用于集成多个子模块，生成顶层文件。

### 11.1 命令

```sh
stk instConn -cfg inst.yaml 
```

### 11.2 inst.yaml示例

```yaml
files:
  - rtl1.sv
  - rtl2.sv
  - rtl3.sv
inc_dirs:
  - './out'
  - './'

defines:
  - ABC1

insts:
  - name: rtl1 u_rtl1 * 2
    conn:
      - axi_slv*    -> axi*<i>
      - p*          -> apb_p*<i>
      - xxds        -> {1'b1, xxds<i>, 1'b0}
      - abc*        -> {abcx*[<i>], 1'b0}
      - '*          -> *_<i>'
  - name: rtl2 u_rtl2
    conn:
      - apb_*       => apb_*
      - axi_mst_*   -> axi_*
  - name: rtl3 u_rtl3
    conn:
      - axi_mst_*   -> axi_*
top: top
out_file: top.sv
logics:
  - logic [3:0][2:0]&[5:0] yyds[0] -> axi_awaddr0 == 12'h123;
```

### 11.2 使用示例

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224150556504-2020241174.png)

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224150638825-257627668.png)

## 12 extFList

extFList是解析带宏定义的filelist的工具，生成vcs/ise等工具支持的filelist

### 12.1 命令

```sh
stk extFList -d POST_SIM src.f -o out.f
```

### 12.2 使用示例

1. 对不存在的文件会报错
2. 对没有使用绝对路径的文件会报错

![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224150941138-1318393510.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224155958980-1479821501.png)
![img](https://img2024.cnblogs.com/blog/898240/202602/898240-20260224151002389-557073553.png)

## 13 yaml语法扩展

yaml语法扩展，支持如下语法：
1. 继承
2. 节点引用
3. 环境变量引用
4. include

## 14 bsim详细介绍


### 1 说明

bsim是用于方便VCS编译和仿真的一个工具，VCS在编译和仿真过程中需要添加众多参数选项，为了减少命令行参选项输入和管理各个选项，bsim工具通过yaml文件清晰呈现选项，并封装功能块为特定选项，方便使用。


各大公司都编写有Makefile，shell脚本，python脚本等工具来方便用户使用。其中Makefile是各大公司使用最多的，且Makefile也是为编译而生，其众多特性都能为编写编译工具提供很多便利。

然而Makefile是一个古老的语言，语法上与众多脚本语言有很大差别，且可调用的函数非常有限，如果要实现复杂的功能，还必须借助shell与来实现，这使得Makefile实现复杂的功能比较难。

python可调用的库非常多，很容易实现复杂的控制功能，但是python配置参数的灵活度不如Makefile。编译工具通常需要非常多需要控制的配置，这些配置的字符串可能还很长，且各个项目，模块需要的配置还千差万别，如果通过参数传递，这可能是灾难性的，对Makefile，通常的做法是在文件的开头添加众多可修改的变量，比如：`opts ?= xxx`
如果opts需要改变，用户有两种修改方式：
1. 调用makefile是通过命令行修改，`make opts=yyy`，这种方式比较灵活，但是没次都得传这个参数
2. 可以直接改Makefile，`opts ?= yyy`，这种方式直接把值固话，省去了每次修改的繁琐，当确少灵活性

为了即能实现复杂功能，又方便配置，bsim采用了python+yaml的方式，将配置和逻辑控制分离，有点类似于web的MVC。
python处理控制，yaml呈现配置，当然python也提供一些命令行参数，提高配置灵活度。


bsim的相比其他脚本，有如下特点：
1. 使用yaml，(支持include,引用，继承等功能)方便选项的管理和扩展，同时支持选项对其覆盖，配置灵活。
2. 将VCS的常用功能做成了开关选项，减少选项输入，封装的功能块有
   1. partcmp
   2. xprop
   3. initreg
   4. ccov
   5. fsdb
   6. gui
3. 提供3种触发编译的模式，（默认/-b/-u）,用户只需要执行sim命令即可完成编译和仿真，非常方便。

### 2. 使用

1. 只仿真

```sh
stk bsim                     # simulation only,
```

> 如果仿真的时候发现没有生成simv，会自动进行编译

2. 编译和仿真
bsim使用分为2步法和3步法：

- 2步法：
```sh
stk bsim -step2 -b           # compile+simulation,
stk bsim -step2 -b -ns       # compile only,
```

- 3步法：
```sh
stk bsim -step3 -a -ns       # analysis(all),
stk bsim -step3 -a -b -ns    # analysis(all)+elaboration,
stk bsim -step3 -a           # analysis(all)+elaboration+simulation,
stk bsim -step3 -a -b        # analysis(all)+elaboration+simulation,
stk bsim -step3 -b -ns       # elaboration
stk bsim -step3 -b           # elaboration+simulation,
stk bsim -step3 -i 2         # analysis(only flist2)+elaboration+simulation(recommend),
stk bsim -step3 -i 2 -ns     # analysis(only flist2)
```

> 执行analysis之前需要一个synopsis_sim.setup的文件，程序会根据flists配置中的lib信息，生成一个synopsis_sim.setup文件，存放在work_dir/exec目录下进行使用。如果需要修改sysnopsis_sim.setup中的内容，可以添加命令到ana.prev_cmd中，修改synopsis_sim.setup

> 调用使用-u选项判断file list更新与否是与前一次使用-u相比的。 即更新检查依赖于上一次的更新检查的结果，会有如下特殊情况。
1. 第一次使用-u选项，必然会认定有更新
2. 调用没有用-u选项命令对有更新的filelist进行编译后，调用使用-u选项命令还是会被认定为有更新，而导致重新编译，因为相对上一次使用-u选项，确实有更新，虽然这个更新已经被编译过，但是没有用-u选项，没有被记录下来。
3. 使用-u选项要有连贯性，要么一直自己跟踪是否有更新，通过默认方式或者-b方式来编译，要么一直使用-u选项

### 3. 配置(yaml和选项)

bsim的配置来源分为2部分，yaml配置文件和bsim的选项

#### 3.1 yaml

> bsim使用的yaml文件相对于标准的yaml文件进行了特性扩展，扩展支持:include、继承、引用、引用环境变量等功能，语法和使用方式参考[yaml语法扩展]()

bsim获取yaml配置分为3种情况：
1. 用`--cfg xxx.yaml`指定了配置文件，且文件存在，那么解析该xxx.yaml，xxx.yaml根节点必须命名为`bsim_cfg`
2. 没有指定了配置文件，当前目录存在bsim_cfg.yaml文件，那么解析该bsim_cfg.yaml，bsim_cfg.yaml根节点必须命名为`bsim_cfg`
3. 即没有指定配置文件，当前目录也不存在bsim_cfg.yaml文件，那么解析stk提供的bsim_base_cfg.yaml配置文件，并把根节点从`bsim_base_cfg`修改为`bsim_cfg`，该方式使用stk的默认配置，适用于临时使用。
<img style="height:400px" src="https://img2023.cnblogs.com/blog/898240/202407/898240-20240710175911904-1053761180.png">
3中方式中，推荐使用第二种，即在仿真目录下存放一个bsim_cfg.yaml文件，这样仿真自动会使用该配置文件。

> 方式1和方式2都会默认在配置文件头插入`!include bsim_base_cfg.yaml`语句，这样用户编写bsim_cfg的时候可以直接继承bsim_base_cfg，进行扩展和修改，减少配置文件编辑量。

配置文件根节点必须为`bsim_cfg`，包含4个子节点，分别为：ana,eac,sim,com，
- ana用于ana命令
- eac用于elab和cmp命令
- sim用于sim命令
- com表示common，即ana/elab/cmp/sim都会使用该部分配置

##### 3.1.1 ana
- vlog_opts：verilog analysis添加的选项
- vhdl_opts：vhdl analysis添加的选项
- prev_cmd：执行analysis前执行的命令，值可以是一条命令，也可以是一个list，包含多个命令
- post_cmd：执行analysis后执行的命令，值可以是一条命令，也可以是一个list，包含多个命令
```yaml
ana:
  prev_cmd: pwd
  post_cmd: 
    - cmd0
    - cmd1
```

##### 3.1.2 eac
- opts: 编译添加的选项
- ccov_opts：如果com.ccov选项打开时，编译添加的选项
- top_hire：验证环境或者设计的顶层名字
- partcmp：分块编译的开关
- partcmp_mode：分块编译的模式，可以选择VCS提供的模式：adaptive_sched/autopart/autopart_low，也可以编写topcfg.v文件来收到设计分块。
- xprop：xprop选项开关
- xprop_mode：xprop的模式，可以选VCS提供的3个模式：vmerge/tmerge/xmerge，也可以选择编写xprop.cfg配置文件
- initreg：initreg选项开关
- initreg_cfg：initreg设置的配置文件
- prev_cmd：执行elaborate和compile前执行的命令，值可以是一条命令，也可以是一个list，包含多个命令
- post_cmd：执行elaborate和compile后执行的命令，值可以是一条命令，也可以是一个list，包含多个命令

##### 3.1.3 sim
- opts：仿真添加的选项
- ccov_opts：com.ccov选项打开时，仿真添加的选项
- tc：设置UVM的执行用例名
- plusarg：如果仿真包含众多plusarg参数，可以将这些参数统一加入到一个文件中，然后将文件添加此处。这用户参数与VCS的选项分离，方便管理。
- rand_seed：是否产生随机种子进行仿真，通常在用例仿真稳定后才开启该选项。
- step3：VCS编译的模式，默认情况下，sim命令引发编译会采用2步法（compile），step3选项开启后，sim命令引发编译会采用3步法（analysis+elaborate）
- prev_cmd：执行simulation前执行的命令，值可以是一条命令，也可以是一个list，包含多个命令
- post_cmd：执行simulation后执行的命令，值可以是一条命令，也可以是一个list，包含多个命令
- print_level: UVM_NONE, UVM_LOW, UVM_MEDIUM, UVM_HIGH, UVM_FULL, UVM_DEBUG

##### 3.1.4 com

- flists：file list, 该配置除了写file list文件，还要说明file list的文件类型，编译的lib存放在那个逻辑库，多个file list可以写成列表，格式为：`type|lib|filelist`，即3部分内容通过"|"分隔，
    - type：可选值为：vlog和vhdl，分别表示filelist的文件为verilog文件和vhdl文件
    - lib：analysis存放结果的位置，如果省略存放在work库中
    - filelist：即file list文件

    三部分内容中，`type`和`lib`可以省略，有如下三中情况：
    - `type|lib|filelist`：不省略
    - `type|filelist`：省略lib，默认为work
    - `filelist`：省略lib，默认为work，省略type，默认为vlog
    > 不省略lib，则type也不能省略

    例子：
    ```yaml
    # 一个file list
    flists: $$env_dir/tb/tb.f # 等效于:vlog|work|$$env_dir/tb/tb.f
    # 多个file list
    flists:
      - vhdl|vh|$$env_dir/tb/vh.f
      - vlog|$$env_dir/tb/abc.f   # 等效于:vlog|work|$$env_dir/tb/abc.f
      - $$env_dir/tb/env.f        # 等效于:vlog|work|$$env_dir/tb/env.f
    ```
    > 多个filelist通常有依赖关系，要把被依赖的filelist写在前面
- flist_define: 解析filelist时需要的宏
- work_dir：工作目录，可以为相对目录，也可以为绝对目录。该目录下会创建：exec/wave/log/verdi/cov/merge等目录，用于存放编译和仿真的文件。其中波形存放在wave目录下，log文件存放在log目录下，覆盖率文件存放在cov目录下，调用merge命令，得到merge覆盖率存放在merge目录下。
- ccov：覆盖率开关，会同时影响编译和仿真
- fsdb：dump波形的开关，，使能该开关会后无效在代码中调用`$fsdbDumpvars`，bsim会生成tcl脚本自动dump波形。
- uvm_en：UVM使能开关，这个为True，编译时候才会编译UVM
- uvm_work：UVM analysis时，lib存放位置
- prev_cmd: 执行bsim前执行的命令
- post_cmd: 执行bsim后执行的命令
- script_file: 在执行命令前执行的脚本文件
- script_args: 脚本文件的参数

简单例子：
```yaml
bsim_cfg(bsim_base_cfg):
  com:
    flists:
      - vhdl|work|$$env_dir/tb/vh.f
      - vlog|work|$$env_dir/tb/abc.f
    fsdb: true
  sim:
     opts+: >-
        +uvm_set_verbosity=\"*,connect_ph,UVM_HIGH,build\"
```
##### 4.1.5 bsim_base_cfg.ymal

```yaml
bsim_base_cfg:
  ana:
    prev_cmd:
    post_cmd:
    vlog_opts: >-
      -full64 -ntb_opts uvm-1.2 -sverilog -lca   # 如果要添加"需要使用反斜杠\"
      -timescale=1ns/1ps
      -ntb_opts check
      +vcs+lic+wait
      -kdb
    vhdl_opts: >-
      -full64 -vhdl93 -kdb

  eac:    # elaborate and compile
    prev_cmd:
    post_cmd:
    opts: >-
      -full64 -ntb_opts uvm-1.2 -sverilog -lca
      -timescale=1ns/1ps
      -ntb_opts check
      +vcs+lic+wait
      -kdb +lint=TFIPC-L
      +error+100
    # -P $$VERDI_HOME/share/PLI/VCS/LINUX64/novas.tab
    # $$VERDI_HOME/share/PLI/VCS/LINUX64/pli.a
    ccov_opts: -cm line+fsm+cond+tgl+assert+branch -cm_tgl mda
    top_hire: top
    partcmp: false
    partcmp_mode: adaptive_sched  #adaptive_sched | autopart | autopart_low | autopart_high | $(ENV_SIM_DIR)/cfg/topcfg.v
    xprop: false
    xprop_mode:  # vmerge | tmerge | xmerge | $(ENV_SIM_DIR)/cfg/xprop.cfg
    initreg: false
    initreg_cfg:

  sim:
    prev_cmd:
    post_cmd:
    opts: >-
      -full64 +notimingcheck -assert nopostproc +vcs+lic+wait
      -assert global_finish_maxfail=10
      +ntb_stop_on_constraint_solver_error=1
      +uvm_set_action="*,RegModel,UVM_WARNING,UVM_NO_ACTION"
    ccov_opts: -cm line+fsm+cond+tgl+assert+branch -cm_tgl mda
    tc: tc001_sanity
    plusarg: # $$(ENV_SIM_DIR)/cfg/args/default.cfg
    rand_seed: True
    step3: True

  com:
    flists:   # ana+elab
    flist_defines: DUMMY_DDR+DUMMY_PCIE
    work_dir: work # ana+elab+sim
    ccov: false  # elab+sim
    fsdb: false  # elab+sim
    uvm_work: work
    uvm_en: True
    prev_cmd:
    post_cmd:
    script_file:
    script_args:
```

#### 3.2 选项

bsim包含很多选项，选择在analisys/elaborate/simulate的不同阶段起作用，把选项分为了多个组
| 命令         | 选项组成                     |
| ------------ | ---------------------------- |
| common       | 在所有流程中会可能用到的选项 |
| elab+cmp     | elab和cmp可能用到的选项      |
| elab+cmp+sim | elab/cmp和sim可能用到的选项  |
| sim          | sim可能用到的选项            |
| action       | 功能选项                     |

##### 3.2.1 common

- -cfg：指定配置的yaml文件，如果不指定会尝试查看当前目录下是否有bsim_cfg.yaml的文件，有的话使用该文件，如果没有则使用默认的bsim_base_cfg.yaml.
- -d/-work_dir：工作目录，该目录下会创建exec/wave/log/cov/merge/verdi等目录，存放编译和仿真的文件。**使用该选项会覆盖掉yaml的work_dir配置。**，默认为work目录
- 添加选项到执行命令中：`-co='+define+ABC'`
  - -ao: 添加verilog的analysis选项
  - -vao: 添加vhdl的analysis选项
  - -co: 添加compiled的选项
  - -eo: 添加elaborate的选项
  - -so: 添加simulation的选项
- -f：设置file list，与yaml中的flists格式一样，分为3部分：type|lib|filelist，其中type,lib可以省略。如果有多个filelist可以通过"+"连接起来。比如：`bsim ana --flists='vhdl|xyz|xyz.f+abc.f'`，analysis的顺序是连接的顺序。**使用该选项会覆盖yaml中flists配置**
> 由于|会被linux识别为管道符，需要flists的值需要用单引号(')，或者双引号包裹(")
- -t：追加yaml中预先写好的选项到命令中，比如：`bsim ana --tag=abc`，那么会将bsim_cfg.ana.abc的值追加到ana命令中。命令只能取特定节点下的tag，具体参考[3.3.4 yaml和选项协同配置(tag)](#tag)
- -uvm：uvm使能，使用该选项，在ana的时候，会独立analysis UVM，同时在cmp和elab的时候添加-ntb_opts uvm-1.2选项。**使用该选项会使能uvm，无论yaml中的uvm_en值是True还是False**
- -D: 设置解析filelist的宏，具体参考[stk.extFList](#stk.extFList)
- -u：设置触发编译的条件为filelist更新，选项与-b互斥。如果为2步法，那么有filelist更新，就会进行compile;如果为3步法，会先对有更新的filelist进行analysis，然后elaborate。
- -sf: script_file: 设置script文件，**使用该选项会覆盖掉yaml文件中配置的script_file**
- -sa: script_args: 脚本参数，**最终的script_args=yaml.script_args + args.script_args**
- -debug: 测试模式
- -dump_cfg: 把bsim配置dump到work_dir目录下

##### 3.2.2 elab_cmp
- -partcmp：分块编译开关，使用该选项，会在compile和elaborate时使用分块编译，分块模式有--partcmp_mode设置。**使用该选项会使能分块编译，无论yaml中的partcmp值是True还是False**
- --partcmp_mode：分块编译的模式，可以选择VCS提供的模式：adaptive_sched/autopart/autopart_low，也可以编写topcfg.v文件来收到设计分块。**使用该选项会覆盖yaml中partcmp_mode配置**
- -xprop：xprop开关，**使用该选项会使能xprop，无论yaml中的xprop值是True还是False**
- -xprop_mode：xprop的模式，可以选VCS提供的3个模式：vmerge/tmerge/xmerge，也可以选择编写xprop.cfg配置文件，**使用该选项会覆盖yaml中xprop_mode配置**

##### 3.2.3 elab_cmp_sim
- -ccov：覆盖率收集开关，**使用该选项会使能覆盖率收集，无论yaml中的ccov值是True还是False**
- -ccov_opts：覆盖率收集选项。覆盖率收集选项在compile/elaborate和simulation都需要设置，且选项不完全一样。
- -initreg：initreg开关，**使用该选项会使能initreg，无论yaml中的initreg值是True还是False**
- -initreg_cfg：设置initreg的配置文件
- -top: 设置top模块
- -gui：启动图形界面
- -fsdb：波形dump开关，使能该开关会后无需在代码中调用`$fsdbDumpvars`，bsim会生成tcl脚本自动dump波形。**使用该选项会dump波形，无论yaml中的fsdb值是True还是False**

> 默认-fsdb会dump top开始的所有层次，如果想修改层次，可以手动编写fsdb.do文件，让后在prev_cmd中把编写的文件拷贝到`work_dir/exec`目录下，bsim会自动执行该文件。

##### 3.2.4 sim_only
- -seed：设置simulation的种子号
- -p：如果仿真包含众多plusarg参数，可以将这些参数统一加入到一个文件中，然后将文件添加此处。这用户参数与VCS的选项分离，方便管理。**使用该选项会覆盖yaml中plugarg配置**
- -pl：print_level设置UVM的打印等级，可选值为：UVM_NONE, UVM_LOW, UVM_MEDIUM, UVM_HIGH, UVM_FULL, UVM_DEBUG
- -rand_seed：种子随机开关，**使用该选项会使能分块编译，无论yaml中的rand_seed值是True还是False**
- -step3：3步法开关，**使用该选项会使能分块编译，无论yaml中的step3值是True还是False**
- -step2：2步法开关
- -tc：设置UVM的执行用例名**使用该选项会覆盖yaml中tc配置**
- -pname: 为仿真log文件名添加额外的标识

##### 3.2.5 action
- -b：强制触发编译，选项与-u互斥。3步法时，没有结合-i选项时，会强制analysis所以filelist，然后elaborate；如果结合-i使用，则只强制analysis -i指定的filelist，然后在elaborate。2步法时，是否使用-i，行为都一样，即强制compile。
- -i：
  - 不与-u结合时：一定编译-i指定的file list
  - 与-u结合时：只查看-i指定的file list是否有更新


#### 3.3 yaml与选项关系

##### 3.3.1 yaml与选项共同配置

yaml和程序选项都可以控制以下配置，其中bsim选项的优先级更高，只有bsim选项没有配置的情况下，才会采用ymal的配置值
- bool类型：
`value = bsim.value or yaml.value`：比如`stk bsim -fsdb`，那么dump fsdb功能打开，无论yaml中配置的fsdb为True还是false
- str类型：
  - bsim添加添加参数：`value = bsim.value`，此时会覆盖yaml的配置值，比如yaml.com.work_dir=work, `stk bsim -work_dir abc`，那么work_dir的配置值为abc
  - bsim不添加参数：`value = yaml.value`

| 选项          | 选项名        | yaml                       | yaml与选项关系 |
| ------------- | ------------- | -------------------------- | -------------- |
| -d            | work_dir      | bsim_cfg.com.work_dir      | arg_ovd_yml    |
| -ao           | ana_opts      | bsim_cfg.ana.vlog_opts     | opt_merge      |
| -avo          | vhdl_ana_opts | bsim_cfg.ana.vhdl_opts     | opt_merge      |
| -co           | cmp_opts      | bsim_cfg.eac.cmp_opts      | opt_merge      |
| -eo           | elab_opts     | bsim_cfg.eac.elab_opts     | opt_merge      |
| -so           | sim_opts      | bsim_cfg.sim.opts          | opt_merge      |
| -f            | flists        | bsim_cfg.com.flists        | list_merge     |
| -uvm          | uvm           | bsim_cfg.com.uvm_en        | arg_xor_yml    |
| -D            | flist_defines | bsim_cfg.com.flist_defines | fl_def_merge   |
| -partcmp      | partcmp       | bsim_cfg.eac.partcmp       | arg_or_yml     |
| -partcmp_mode | partcmp_mode  | bsim_cfg.eac.partcmp_mode  | arg_ovd_yml    |
| -xprop        | xprop         | bsim_cfg.eac.xprop         | arg_or_yml     |
| -xprop_mode   | xprop_mode    | bsim_cfg.eac.xprop_mode    | arg_ovd_yml    |
| -cov          | cov           | bsim_cfg.com.cov           | arg_or_yml     |
| -scov_opts    | scov_opts     | bsim_cfg.sim.cov_opts      | arg_ovd_yml    |
| -ccov_opts    | ccov_opts     | bsim_cfg.eac.cov_opts      | arg_ovd_yml    |
| -initreg      | initreg       | bsim_cfg.eac.initreg       | arg_or_yml     |
| -initreg_cfg  | initreg_cfg   | bsim_cfg.eac.initreg_cfg   | arg_ovd_yml    |
| -top          | top           | bsim_cfg.eac.top_hire      | arg_ovd_yml    |
| -fsdb         | fsdb          | bsim_cfg.com.fsdb          | arg_or_yml     |
| -p            | plusarg       | bsim_cfg.sim.plusarg       | list_merge     |
| -pl           | print_level   | bsim_cfg.sim.print_level   | arg_ovd_yml    |
| -rand_seed    | rand_seed     | bsim_cfg.sim.rand_seed     | arg_or_yml     |
| -step3        | step3         | bsim_cfg.sim.step3         | arg_or_yml     |
| -step2        | step2         | bsim_cfg.sim.step3         | arg_not        |
| -tc           | tc            | bsim_cfg.sim.tc            | arg_ovd_yml    |
| -sf           | script_file   | bsim_cfg.com.script_file   | arg_ovd_yml    |
| -sa           | script_args   | bsim_cfg.com.script_args   | sa_merge       |

- arg_ovd_yml: 添加的选项会覆盖yaml的配置值
- opt_merge: 添加的选项会与yaml的配置值进行合并
- list_merge: 添加的选项会与yaml的配置值进行合并
- arg_xor_yml: 添加的选项会与yaml的配置值进行异或
- fl_def_merge: 添加的选项会与yaml的配置值进行合并 
- arg_or_yml: 添加的选项会与yaml的配置值进行或
- arg_not: 选项的值取反
- sa_merge: 添加的选项会与yaml的配置值进行合并，如果选项的`-sa name=value`与yaml的`script_args: name=value`有相同的name，那么-sa覆盖yaml的值

##### 3.3.2 选项独有配置
| 选项   | 选项名   |
| ------ | -------- |
| -gui   | gui      |
| -seed  | seed     |
| -pname | pname    |
| -i     | idx      |
| -a     | ana_en   |
| -b     | build_en |
| -ns    | not_sim  |

##### 3.3.3 yaml独有配置

- com.uvm_work


##### 3.3.4 yaml和选项协同配置<a name="tag">(tag)</a>

在yaml添加tag(就是字典)，并通过-t选项来引用，将tag的值添加到执行的命令中，从而减少命令行输入的字符数量。
- yaml在ana/elab/sim下面添加`tag_name: tag_options`，
- 在选项中使用`cmd -t tag_name`，-t可以多次使用，且一次可以添加多个tag_name,比如`sim -t tag_name1+tag_name2 -t tag_name3`，

举例：
yaml:
```yaml
bsim_cfg:
  eac:
    debug: -debug_access+all
```
命令：
```sh
stk bsim -t debug
```

通过`-t debug`往命令中添加了`-debug_access+all`选项，这种方式在添加的选项比较长的时候比较有优势，如果短可以直接使用`-ao/co/eo/so`选项

> 注意：tag引用节点位置与命令有对应关系。

| 命令 | 可以引用的yaml节点        |
| ---- | ------------------------- |
| ana  | bsim_cfg.ana/bsim_cfg.com |
| elab | bsim_cfg.eac/bsim_cfg.com |
| cmp  | bsim_cfg.eac/bsim_cfg.com |
| sim  | bsim_cfg.sim/bsim_cfg.com |

##### 3.3.5 script

yaml和bsim选项都可以配置script_file和script_args
- script_file: 如果bsim选项没有配置script_file，那么使用yaml的配置，否则使用bsim选项的配置
- script_args: yaml的配置与bsim选项的配置合并，如果yaml和选项有同名的配置，那么选项的配置覆盖yaml的配置

比如：yaml有`script_args: post_sim=False,abc=123`，bsim选项有`script_args: post_sim=True`，那么最终的script_args为`post_sim=True,abc=123`

script约束：
1. 不能import其他文件的内容
2. 可以import python标准库,re等常用库
3. 可以包含多个function，必须包含main函数，main函数的原型为：`main(bsim_cfg)`，其中bsim_cfg为yaml结构的对象，可以读取和修改yaml任何节点的值

```py
def main(bsim_cfg):
    print(bsim_cfg)
    print(bsim_cfg['com']['work_dir'])
    bsim_cfg['com']['work_dir'] = 'debug'
    for name, value in bsim_cfg['com']['script_args']:
      print(name, value)
    if bsim_cfg['script_args']['post_sim']:
      bsim_cfg['com']['flist_defines'] += '+POST_SIM'
```

### 4. 规则

#### 4.1 log名字和路径

路径：所以log都存放在work_dir/log目录下
名字:
1. analysis的log文件名与filelist有关，logname={filelist_name}{filelist_idx}_an.log
2. compile的log文件名固定为cmp.log
3. elaborate的log文件名固定为elab.log
4. simulation的log文件名与tc,seed,pname值有关，logname={tc_name}_{pname}_{seed}.log，seed默认为12345678
