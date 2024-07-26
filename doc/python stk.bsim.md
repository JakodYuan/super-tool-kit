# stk.bsim

[TOC]

## 1 说明

bsim是用于方便VCS编译和仿真的一个工具，VCS在编译和仿真过程中需要添加众多参数选项，为了减少参数输入，各大公司都编写有Makefile，shell脚本，python脚本等工具来方便用户使用。其中Makefile是各大公司使用最多的，且Makefile也是为编译而生，其众多特性都能为编写编译工具提供很多便利。

然而Makefile是一个古老的语言，语法上与众多脚本语言有很大差别，且可调用的函数非常有限，如果要实现复杂的功能，还必须借助shell与来实现，这使得Makefile实现复杂的功能比较难。

python可调用的库非常多，很容易实现复杂的控制功能，但是python配置参数的灵活度不如Makefile。编译工具通常需要非常多需要控制的配置，这些配置的字符串可能还很长，且各个项目，模块需要的配置还千差万别，如果通过参数传递，这可能是灾难性的，对Makefile，通常的做法是在文件的开头添加众多可修改的变量，比如：`opts ?= xxx`
如果opts需要改变，用户有两种修改方式：
1. 调用makefile是通过命令行修改，`make opts=yyy`，这种方式比较灵活，但是没次都得传这个参数
2. 可以直接改Makefile，`opts ?= yyy`，这种方式直接把值固话，省去了每次修改的繁琐，当确少灵活性

为了即能实现复杂功能，又方便配置，bsim采用了python+yaml的方式，将配置和逻辑控制分离，有点类似于web的MVC。
python处理控制，yaml呈现配置，当然python也提供一些命令行参数，提高配置灵活度。

bsim的相比其他脚本，有如下优点：
1. 使用yaml，方便选项的管理和扩展，同时支持python选项对其覆盖，配置灵活。
2. 将VCS的常用功能做成了开关选项，减少选项输入，封装的功能块有
   1. partcmp
   2. xprop
   3. initreg
   4. ccov
   5. fsdb
   6. gui
3. 提供3种触发编译的模式，（默认/-b/-u）,用户只需要执行sim命令即可完成编译和仿真，非常方便。

## 2. 使用

bsim分为7个子功能：ana/elab/cmp/sim/verdi/dve/merge

### 2.1 ana/elab/cmp/sim

ana->elab->sim之间有依赖关系，cmp->sim也有依赖关系。这两条关系链，在-b/-u/-i idx选项下的行为会有差异。
-b/-u是互斥选项，不能同时使用，-i可以分别于-b和-u结合
设计考虑：
- 默认：能保证当前执行时，不往前级执行，但是当前一定执行。
- -b：build，如果有前级，前级全部执行，当前也一定执行。
- -u：update，如果有前级，只有前级有更新时执行前级。当前不一定执行。
- -i：index，指定执行的file list, 前级执行只考虑`-i idx`指定的file list，不使用-i选项的时候`idx==-1`，使用-i选项后`idx!=-1`

#### 2.1.1 ana

- `stk bsim ana`:ana(build=False,update=False,idx=x)
    - idx==-1：对所有filelist进行ana
    - idx!=-1：对idx=x的filelist进行ana
- `stk bsim ana -b`:ana(build=True,update=False,idx=x)
    - 与上面相同
- `stk bsim ana -u`:ana(build=False,update=True,idx=x)
    - idx==-1：检查所以filelist是否有更新，对有更新的进行ana
    - idx!=-1：检查idx=x的filelist是否有更新，如果有更新则进行ana，否则啥也不做


#### 2.1.2 elab

- `stk bsim elab`:elab(build=False,update=False,idx=x)
    - idx==-1:
        - elab需要的lib都存在，则直接进行elab
        - elab需要的lib=a不存在，先调用ana(build=False,update=False,idx=a.idx)，然后进行elab
    - idx!=-1:
        - elab需要的lib都存在，先调用ana(build=False,update=False,idx=x)，然后进行elab
        - elab需要的lib=a不存在，先调用ana(build=False,update=False,idx=a.idx)和ana(build=False,update=False,idx=x)，然后进行elab
- `stk bsim elab -b`:elab(build=True,update=False,idx=x)
    - idx==-1，对所有filelist进行ana，然后进行elab
    - idx!=-1，对idx指定的filelist和lib不存在的filelist进行ana，然后进行elab
        - elab需要的lib都存在：先调用ana(build=False,update=False,idx=x)，然后进行elab
        - elab需要的lib=a不存在：先调用ana(build=False,update=False,idx=a.idx)和ana(build=False,update=False,idx=x)，然后进行elab
- `stk bsim elab -u`:elab(build=False,update=True,idx=x)
    - 先调用`has_ana=ana(build=False,update=True,idx=x)`
    - has_ana=False+simv已存在：啥也不做
    - has_ana=False+simv不存在：elab
    - has_ana=True+simv已存在：elab
    - has_ana=True+simv不存在：elab


#### 2.1.3 cmp

- `stk bsim cmp`:cmp(build=False,update=False,idx=x)
    - 直接进行cmp，
- `stk bsim cmp -b`:cmp(build=True,update=False,idx=x)
    - 直接进行cmp
- `stk bsim cmp -u`:cmp(build=False,update=True,idx=x)
    - idx!=-1时，只检查idx指定的filelist
    - idx==-1时，检查所以filelist
    - filelist有更新+simv已存在：啥也不做
    - filelist有更新+simv不存在：cmp
    - filelist无更新+simv已存在：cmp
    - filelist无更新+simv不存在：cmp

#### 2.1.4 sim 

- `stk bsim sim`
    - idx!=-1:
        - 先调用cmp(build=False, update=False,idx=x)或者elab(build=False, update=False,idx=x)
        - 然后执行仿真
    - idx==-1:
        - simv存在则直接仿真
        - simv不存在，则根据step3分别调用elab(build=False,update=False,idx=-1)或者cmp(build=False,update=False,idx=-1)，然后执行仿真
- `stk bsim sim -b`
    - 先调用elab(build=True,update=False,idx=x)或者cmp(build=True,update=False,idx=x)进行编译
    - 然后执行仿真
- `stk bsim sim -u`
    - 先调用elab(build=False,update=True,idx=x)或者cmp(build=False,update=True,idx=x)进行编译
    - 然后执行仿真

> 调用使用-u选项判断file list更新与否是与前一次使用-u相比的。 即更新检查依赖于上一次的更新检查的结果，会有如下特殊情况。
1. 第一次使用-u选项，必然会认定有更新
2. 调用没有用-u选项命令对有更新的filelist进行编译后，调用使用-u选项命令还是会被认定为有更新，而导致重新编译，因为相对上一次使用-u选项，确实有更新，虽然这个更新已经被编译过，但是没有用-u选项，没有被记录下来。
3. 使用-u选项要有连贯性，要么一直自己跟踪是否有更新，通过默认方式或者-b方式来编译，要么一直使用-u选项

> `cmd -b`与`cmd`两种的区别（cmd in [ana,elab,cmp,sim]）
1. 在--idx!=-1情况下，`cmd -b`与`cmd`功能完全相同，会执行必须的和idx指定的编译
2. 在--idx==-1情况下，`cmd -b`会进行所以的编译，`cmd`只执行必须的编译

#### 2.1.4 使用推荐

推荐通过执行sim命令，至于ana/elab和cmp依赖脚本自动推断执行。

三步法：
1. 查询全部file list时间短，无脑采用：`sim -u`
2. 查询特定file list时间短，且只会修改该file list中的文件，无脑采用：`sim -u -i idx`
3. 查询file list比较耗时，且知道那个file list中的文件有修改，根据是否修改选择执行`sim`和`sim -i idx`
4. 就一个file list，且知道是否有修改，自己根据是否有修改选择使用`sim`或者`sim -b`
两步法：
1. 查询全部file list时间短，无脑采用：`sim -u`
2. 查询特定file list时间短，且只会修改该file list中的文件，无脑采用：`sim -u -i idx`
3. 知道是否有修改，根据是否有修改选择调用`sim`和`sim -b`

如果只是想编译，则可以执行`cmp`或者`ana+elab`

### 2.2 verdi/dve/merge 

#### 2.2.1 verdi

- `bsim verdi`：
  - 打开当前工作空间的verdi，verdi会显示代码
  - 等效于`verdi -dbdir work_dir/exec/simv.daidir` 
- `bsim -wave xx.fsdb`
  - 打开带波形的verdi
  - 等效于`verdi -ssf xxx.fsdb` 

#### 2.2.2 dve
- `bsim dve`：
  - 打开当前工作空间的dve，dve包含当前的覆盖信息
  - 等效于`dve -covdir work_dir/cov/simv*` 

#### 2.2.3 merge

- `bsim merge`
  - 将work_dir/ccov/目录下的覆盖信息与work_dir/merge/下的覆盖信息进行merge，结果保存在work_dir/merge目录下
  - 等效于`urg -dir work_dir/cov/*.vdb -dir work_dir/merge/*.vdb -dbname work_dir/merge/merge_time/test -elfile elfile`


## 3. 配置(yaml和选项)

bsim的配置来源分为2部分，yaml配置文件和bsim的选项

### 3.1 yaml

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
verdi/dve/merge没有添加相应的配置节点，用户有需要可以自行添加。

#### 3.1.1 ana
- vlog_opts：verilog analysis添加的选项
- vhdl_opts：vhdl analysis添加的选项

#### 3.1.2 eac
- opts: 编译添加的选项
- ccov_opts：如果com.ccov选项打开时，编译添加的选项
- top_hire：验证环境或者设计的顶层名字
- partcmp：分块编译的开关
- partcmp_mode：分块编译的模式，可以选择VCS提供的模式：adaptive_sched/autopart/autopart_low，也可以编写topcfg.v文件来收到设计分块。
- xprop：xprop选项开关
- xprop_mode：xprop的模式，可以选VCS提供的3个模式：vmerge/tmerge/xmerge，也可以选择编写xprop.cfg配置文件
- initreg：initreg选项开关
- initreg_cfg：initreg设置的配置文件

#### 3.1.3 sim
- opts：仿真添加的选项
- ccov_opts：com.ccov选项打开时，仿真添加的选项
- tc：设置UVM的执行用例名
- plusarg：如果仿真包含众多plusarg参数，可以将这些参数统一加入到一个文件中，然后将文件添加此处。这用户参数与VCS的选项分离，方便管理。
- rand_seed：是否产生随机种子进行仿真，通常在用例仿真稳定后才开启该选项。
- step3：VCS编译的模式，默认情况下，sim命令引发编译会采用2步法（compile），step3选项开启后，sim命令引发编译会采用3步法（analysis+elaborate）

#### 3.1.4 com

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
- work_dir：工作目录，可以为相对目录，也可以为绝对目录。该目录下会创建：exec/wave/log/verdi/cov/merge等目录，用于存放编译和仿真的文件。其中波形存放在wave目录下，log文件存放在log目录下，覆盖率文件存放在cov目录下，调用merge命令，得到merge覆盖率存放在merge目录下。
- ccov：覆盖率开关，会同时影响编译和仿真
- fsdb：dump波形的开关，，使能该开关会后无效在代码中调用`$fsdbDumpvars`，bsim会生成tcl脚本自动dump波形。
- uvm_en：UVM使能开关，这个为True，编译时候才会编译UVM
- uvm_work：UVM analysis时，lib存放位置


#### 4.1.5 bsim_base_cfg.ymal

```yaml
bsim_base_cfg:
  ana:
    vlog_opts: >-
      -full64 -ntb_opts uvm-1.2 -sverilog -lca
      -timescale=1ns/1ps
      -ntb_opts check
      +vcs+lic+wait
      -kdb
    vhdl_opts: >-
      -full64 -vhdl93 -kdb

  eac:    # elaborate and compile
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
    work_dir: work # ana+elab+sim
    ccov: false  # elab+sim
    fsdb: false  # elab+sim
    uvm_work: work
    uvm_en: True
```

### 3.2 选项

bsim包含多个命令，命令的选项有共同的部分，也有特有的部分，分为如下6部分，每个命令的选项有这6部分的一个或多个部分组成。
| 命令  | 选项组成                              |
| ----- | ------------------------------------- |
| ana   | common                                |
| elab  | common+elab_cmp+elab_cmp_sim          |
| cmp   | common+elab_cmp+elab_cmp_sim          |
| sim   | common+elab_cmp+elab_cmp_sim+sim_only |
| verdi | common+verdi_only                     |
| dve   | common                                |
| merge | common+merge_only                     |

#### 3.2.1 common

- --cfg：指定配置的yaml文件，如果不指定会尝试查看当前目录下是否有bsim_cfg.yaml的文件，有的话使用该文件，如果没有则使用默认的bsim_base_cfg.yaml.
- --work_dir：工作目录，该目录下会创建exec/wave/log/cov/merge/verdi等目录，存放编译和仿真的文件。**使用该选项会覆盖掉yaml的work_dir配置。**
- --opts：追加选项到执行的命令中，只追加到当前要执行的命令中，不会最近到触发的命令中。比如：`bsim sim --opts="+define+ABC"`，由于sim命令触发了cmp命令，cmp命令不会追加+define+ABC的选项，只有sim命令会追加该选项。
- --flists：设置file list，与yaml中的flists格式一样，分为3部分：type|lib|filelist，其中type,lib可以省略。如果有多个filelist可以通过"+"连接起来。比如：`bsim ana --flists='vhdl|xyz|xyz.f+abc.f'`，analysis的顺序是连接的顺序。**使用该选项会覆盖yaml中flists配置**
> 由于|会被linux识别为管道符，需要flists的值需要用单引号(')，或者双引号包裹(")
- --idx/-i：
  - 不与-u结合时：一定编译--idx指定的file list
  - 与-u结合时：只查看--idx指定的file list是否有更新
- --tag/-t：追加yaml中预先写好的选项到命令中，比如：`bsim ana --tag=abc`，那么会将bsim_cfg.ana.abc的值追加到ana命令中。命令只能取特定节点下的tag，具体参考[3.3.4 yaml和选项协同配置(tag)](#tag)
- -gen_setup：在3步法的时候，编译需要一个synopsis_sim.setup的文件，使用该选项会根据flists配置中的lib信息，生成一个synopsis_sim.setup文件，放在仿真目录下。编译的时候，会拷贝该文件到work_dir/exec目录下进行使用，所以如果要手动编写synopsis_sim.setup文件也应该放在仿真目录下。
- -uvm：uvm使能，使用该选项，在ana的时候，会独立analysis UVM，同时在cmp和elab的时候添加-ntb_opts uvm-1.2选项。**使用该选项会使能uvm，无论yaml中的uvm_en值是True还是False**
- --update/-u：设置触发编译的条件为filelist更新，选项与-b互斥。如果为2步法，那么有filelist更新，就会进行compile;如果为3步法，会先对有更新的filelist进行analysis，然后elaborate。具体细节参考 [2.1 ana/elab/cmp/sim](#21-anaelabcmpsim)
- --build/-b：强制触发编译，选项与-u互斥。3步法时，没有结合--idx选项时，会强制analysis所以filelist，然后elaborate；如果结合--idx使用，则只强制analysis idx指定的filelist，然后在elaborate。2步法时，是否使用--idx，行为都一样，即强制compile。具体细节参考 [2.1 ana/elab/cmp/sim](#21-anaelabcmpsim)

#### 3.2.2 elab_cmp
- -partcmp：分块编译开关，使用该选项，会在compile和elaborate时使用分块编译，分块模式有--partcmp_mode设置。**使用该选项会使能分块编译，无论yaml中的partcmp值是True还是False**
- --partcmp_mode：分块编译的模式，可以选择VCS提供的模式：adaptive_sched/autopart/autopart_low，也可以编写topcfg.v文件来收到设计分块。**使用该选项会覆盖yaml中partcmp_mode配置**
- -xprop：xprop开关，**使用该选项会使能xprop，无论yaml中的xprop值是True还是False**
- --xprop_mode：xprop的模式，可以选VCS提供的3个模式：vmerge/tmerge/xmerge，也可以选择编写xprop.cfg配置文件，**使用该选项会覆盖yaml中xprop_mode配置**

#### 3.2.3 elab_cmp_sim
- -ccov：覆盖率收集开关，**使用该选项会使能覆盖率收集，无论yaml中的ccov值是True还是False**
- --ccov_opts：覆盖率收集选项。覆盖率收集选项在compile/elaborate和simulation都需要设置，且选项不完全一样。
- -initreg：initreg开关，**使用该选项会使能initreg，无论yaml中的initreg值是True还是False**
- --initreg_cfg：设置initreg的配置文件
- -gui：启动图形界面
- -fsdb：波形dump开关，使能该开关会后无效在代码中调用`$fsdbDumpvars`，bsim会生成tcl脚本自动dump波形。**使用该选项会dump波形，无论yaml中的fsdb值是True还是False**

#### 3.2.4 sim_only
- --seed：设置simulation的种子号
- --plusarg：如果仿真包含众多plusarg参数，可以将这些参数统一加入到一个文件中，然后将文件添加此处。这用户参数与VCS的选项分离，方便管理。**使用该选项会覆盖yaml中plugarg配置**
- --print_level：设置UVM的打印等级，可选值为：UVM_NONE, UVM_LOW, UVM_MEDIUM, UVM_HIGH, UVM_FULL, UVM_DEBUG
- -rand_seed：种子随机开关，**使用该选项会使能分块编译，无论yaml中的rand_seed值是True还是False**
- -step3：3步法开关，**使用该选项会使能分块编译，无论yaml中的step3值是True还是False**
- --tc：设置UVM的执行用例名**使用该选项会覆盖yaml中tc配置**

#### 3.2.5 verdi_only
- --wave：设置打开的波形

#### 3.2.6 merge_only
- --elfile：设置覆盖率的exclude

### 3.3 yaml与选项关系

#### 3.3.1 yaml与选项共同配置

yaml和程序选项都可以控制以下配置，其中bsim选项的优先级更高，只有bsim选项没有配置的情况下，才会采用ymal的配置值
- bool类型：
  不是添加bsim选项，配置值为yaml的配置值，添加bsim选项，配置值为True.比如:`bsim sim -fsdb`,无论yaml配置是什么，fsdb配置结果都为True
- str类型：
  不添加bsim选项，配置值为yaml的配置值，添加bsim选项，则配置值为bsim选项的值。比如:`bsim sim --work_dir abc`,yaml配置的`work_dir: work`，最后的work_dir的配置值为abc

| yaml配置              | bsim选项                  | 类型                        |
| --------------------- | ------------------------- | --------------------------- |
| ymal.com.flists       | ana/elab/cmp/sim.flists   | str(file list)              |
| ymal.com.work_dir     | ana/elab/cmp/sim.work_dir | str(path)                   |
| yaml.com.ccov         | elab/cmp/sim.ccov         | bool                        |
| yaml.com.fsdb         | elab/cmp/sim.fsdb         | bool                        |
| yaml.com.uvm_en       | ana/elab/cmp.uvm          | bool                        |
| yaml.eac.ccov_opts    | elab/cmp.ccov_opts        | str(options)                |
| yaml.eac.initreg      | elab/cmp.initreg          | bool                        |
| yaml.eac.initreg_cfg  | elab/cmp.initreg_cfg      | str(configure file)         |
| yaml.eac.partcmp      | elab/cmp.partcmp          | bool                        |
| yaml.eac.partcmp_mode | elab/cmp.partcmp_mode     | str(mode or configure file) |
| yaml.eac.xprop        | elab/cmp.xprop            | bool                        |
| yaml.eac.xprop_mode   | elab/cmp.xprop_mode       | str(mode or configure file) |
| yaml.eac.top_hire     | elab/cmp.top              | str(hire)                   |
| yaml.sim.step3        | sim.step3                 | bool                        |
| yaml.sim.plugarg      | sim.plugarg               | file                        |
| yaml.sim.tc           | sim.tc                    | str(test case name)         |
| yaml.sim.rand_seed    | sim.rand_seed             | bool                        |
| yaml.sim.ccov_opts    | sim.ccov_opts             | str(options)                |

> ccov_opts选项，会同时影响编译的ccov_opts和仿真的ccov_opts，sim命令使用该选项时，不能触发编译。

#### 3.3.2 yaml独立配置

下面这些yaml配置不受选项控制

- ana.vlog_opts
- ana.vhdl_opts
- elab.opts
- sim.opts
- com.uvm_work

#### 3.3.3 选项独立配置

下面这些选项与yaml配置无关

- ana.opts
- elab.opts
- cmp.opts
- sim.opts
- ana/elab/cmp/sim.build
- ana/elab/cmp/sim.update
- ana/elab/cmp/sim.idx
- elab/cmp/sim.gui
- sim.seed
- sim.print_level
- verdi.opts
- verdi.wave
- dve.opts
- dve.elfile
- merge.opts

#### 3.3.4 yaml和选项协同配置<a name="tag">(tag)</a>

在yaml添加tag(就是字典)，并通过--tag选项来引用，将tag的值添加到执行的命令中，从而减少命令行输入的字符数量。
- yaml在ana/elab/sim下面添加`tag_name: tag_options`，
- 在选项中使用`cmd --tag tag_name`，--tag可以多次使用，且一次可以添加多个tag_name,比如`sim --tag tag_name1+tag_name2 --tag tag_name3`，

举例：
yaml:
```yaml
bsim_cfg:
  eac:
    debug: -debug_access+all
```
命令：
```sh
stk bsim cmp --tag debug
```

通过`--tag debug`往命令中添加了`-debug_access+all`选项，这种方式在添加的选项比较长的时候比较有优势，如果短可以直接使用`--opts`选项

> 注意：tag引用节点位置与命令有对应关系。

| 命令  | 可以引用的yaml节点        |
| ----- | ------------------------- |
| ana   | bsim_cfg.ana/bsim_cfg.com |
| elab  | bsim_cfg.eac/bsim_cfg.com |
| cmp   | bsim_cfg.eac/bsim_cfg.com |
| sim   | bsim_cfg.sim/bsim_cfg.com |
| verdi | bsim_cfg.ana              |
| dve   | bsim_cfg.dve              |
| merge | bsim_cfg.merge            |




