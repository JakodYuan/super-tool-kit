# stk.regress

在芯片验证过程中，会编写随机用例，通过大量回归来找到设计中潜在的BUG。用例回归其实是一个很简单的问题，就是让切换种子，让用例不断仿真，并对仿真出错的用例进行分析。

这个回归工具，根据配置支持自动切换种子，然后并行执行用例，回归的信息状态信息（pass，fail，finish）可以在konsle中实时显示，回归用例的具体信息（比如哪条用例出错了，错误原因是什么，种子号是多少）可以通过输出的log文件，或者使用stk.regrServer在网页中查看，推荐使用网页查看。

回归工具，读取yaml文件得到编译和仿真的参数，编辑简单，可读性也非常高。用例执行支持LSF提交，也支持本地的多进程执行。

[TOC]

## 1 简单使用

下载stk工具后，无论在windows还是在linux中使用，为了保证在任何目录下都可以直接调用stk工具，需要将将stk工具存放目录添加到PATH中。
- windows:略
- linux:在.cshrc中添加如下代码，并进行source
```sh
setenv PATH $PATH\:xxx/stk_dir
setenv PATH `echo $PATH |sed 's/:/\n/g' | sort | uniq | tr -s '\n' ':' | sed 's/:$//g'`
```
> 不能通过alias方式，因为在工具中通过多进程调用`stk parseLog`会新开shell，alias会失效。

测试：在任何目录下输入`stk -h`

使用步骤分为2步：

1. 在执行回归前，必须先有一个yaml格式的回归配置文件，默认为当前目录下的regr.yaml文件，格式可以参考[配置文件](#配置文件)
  - run_cmd：只要在命令行可以只的命令都可以写在run_cmd中，通常编译和仿真会用到大量的VCS仿真工具的选项，为了减少命令行输入选项的数量，各个公司都会有Makefile/python等命令封装脚本，run_cmd的命令也可以调用这些脚本。
  - parse_prog：这个为空或者null/None不会产生log解析job，如果要解析，应该填`stk parseLog`，暂时不支持第三方的log解析工具。parseLog的具体细节可参考[stk.parseLog]()
2. 在命令行中执行stk工具`stk regress`，默认情况下terminal只打印jobgroup的执行结果
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240626155610155-296255675.png)
3. 如果想在terminal打印job的执行结果和job命令信息，可以添加`-show_job_info`，如果想打印job/jobgroup开始执行的信息，可以添加`-show_start_info`
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240626154935261-1903718734.png)

## 2 配置文件

回归的配置的和要执行的任务信息通过一个yaml格式的配置文件承载，该文件能很简洁直观的方式编写和阅读。
为了提高ymal的复用性和扩展性，在基本的yaml语法功能基础上，新增了：include，继承，引用节点，引用环境变量等功能，具体参考[yaml语法扩展]()

### 2.1 文件结构

配置文件包含3个根节点，regr_cfg、builds和simulations
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240626160231907-1363669720.png)
- regr_cfg：为回归的相关配置信息，同时包含了pre_cmd和post_cmd，用于在builds和simlations前后执行。
- builds：用于放置编译的相关配置
- simulations：用于放置仿真相关配置

当然，配置文件为yaml格式的文件，并不约束添加其他的根节点，比如添加`build_common:&build_common`和`sim_common:&sim_common`作为锚点，将build和simulation中的通用配置添加在里边，builds和simulations的下面的job_group直接引用这两个锚点就可以了。

regr_cfg/builds/simulations这3个节点必须包含一些特殊对象，且对象的值为如下几种类型：
1. scalar
   某个对象的值必须为纯量（string,数值等，不能为dict或者list），某些对象会对纯量值的类型有约束，比如max_run_num必须为整数类型
2. list-scalar
   某个对象的值可以为纯量或者数组，但是数组的成员必须为纯量。
   无论对象的值是否是数组，获取对象的值都会得到一个数组
   - 对象值为list：则直接返回该list
   - 对象值为scalar：则返回[scalar]
3. join_list-scalar
   某个对象的值可以为纯量或者数组，但是数组的成员必须为纯量。
   无论对象的值是否是数组，获取对象的值都会得到一个纯量（string）
   - 对象值为scalar：则直接返回scalar
   - 对象值为list：则返回`''.join(list)`
4. list-join_list-scalar
   某个对象的值可以为纯量或者数组，数组的值可以为纯量，也可以为数组，但是二级数组的值必须为纯量。
   无论对象的值是纯量，一维数组还是二维数组，获取该对象的值都会得到一个一维数值。
   - 对象值为纯量：返回[scalar]
   - 对象值为list：返回该list
   - 对象为list-list：返回`[''.join(x) for x in list-list]`，即数组的第二维通过join的方式合并成一个值，第二维允许是稀疏的。如下：
   ```yaml
   list-join_list-scalar:
      - abc
      -
        - xyz
        - www
   ```

### 2.2 regr_cfg

regr_cfg必须包含如下对象

| 对象名                 | 值类型                | 说明                                                                                                                                                                                                                                 |
| ---------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| proj_name              | scalar                | 项目名称                                                                                                                                                                                                                             |
| regr_name              | scalar                | 回归名称，推荐使用模块的名字                                                                                                                                                                                                         |
| database_en            | scalar&boolean        | 只能填`False`和`True`，表示是否使用数据库                                                                                                                                                                                            |
| database               | scalar                | 数据库的路径和名字，这里使用的是Sqlite数据库，如果这个数据库名字不存在，会自动创建                                                                                                                                                   |
| timeout                | scalar&float          | 表示job超时时间，单位为分钟，有误差值为update_interval                                                                                                                                                                                                          |
| max_run_num            | scalar&int            | job_group并行执行的数量，如果job_group中的job任务不是bsub任务，则还会受到max_process_num约束。如果有max_run_num=100,max_process_num=10,如果全部job都是非bsub的，那么并行度只能为10。如果非bsub的job少于10，那么并行度还是可以达到100 |
| max_process_num        | scalar&int            | job_group执行非bsub任务的时候，采用了多进程，该配置约束最大提交的进程数量。这些进程都只能在本地机器上执行，所以进程数量不宜过大，避免导致本地机器宕机                                                                                |
| pre_cmd                | list-join_list-scalar | 在执行build之前执行的命令，可以有多条命令(即pre_cmd为list),每天命令还可以写多行(即list-list)，也可以为空/null/None表示没有要执行的命令                                                                                                                                         |
| post_cmd               | list-join_list-scalar | 在simulation后执行的命令，格式与pre_cmd相同                                                                                                                                                                                          |
| log_dir               | scalar | 存放log文件的目录，log_dir + job_group.log得到一个log文件的全路径,用于网页中读取log文件显示 |
| lsfs                   | dict                  | 这个是一个推荐对象，可以写成dict,每个成员为一个bsub类型，比如:short,long，low_memory,high_memory等，方便在simulation和build中引用                                                                                                    |
| parse_cfg              | dict                  | 包含3个对象，must_contain，can_contain,not_contain，这个对象会添加锚点，以在builds和simulation中引用                                                                                                                                 |
| parse_cfg.must_contain | join_list-scalar      | 用于paserLog解析的参数，表必须包含的关键词句，多个关键词句通过","分隔，没有关键词句可以写`""`或者`null/None`，具体说明参考regress.parseLog                                                                                                 |
| parse_cfg.can_contain  | join_list-scalar      | 用于paserLog解析的参数，表可以包含的关键词句，多个关键词句通过","分隔 ，没有关键词句可以写`""`或者`null/None`，具体说明参考regress.parseLog                                                                                                |
| parse_cfg.not_contain  | join_list-scalar      | 用于paserLog解析的参数，表不能包含的关键词句，多个关键词句通过","分隔 ，没有关键词句可以写`""`或者`null/None`，具体说明参考regress.parseLog                                                                                                |

### 2.3 builds

builds中可以包含多个job_group，每个job_group对象必须包含如下对象

| 对象名       | 值类型                | 说明                                                                                                                                                                                           |
| ------------ | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| run_cmd      | list-join_list-scalar | 要执行的命令，格式与pre_cmd相同                                                                                                                                                                |
| log          | list-value            | log文件的路径和名字，可以有多个，工具会自动根据log文件的个数和must/can/not_contain参数，产生对应数量的log解析命令，添加到run_cmd列表的末尾。如果想不进行log解析，可以将log的值填成None或者Null |
| lsf          | scalar                | None或者bsub命令，如果为bsub命令，推荐引用regr_cfg.lsfs下的值，如果job_group命令在本地机器上通过多进程执行，则填None                                                                           |
| must_contain | list-value            | 同这里的值推荐引用regr_cfg.parse_cfg的值                                                                                                                                                       |
| can_contain  | list-value            | 同上                                                                                                                                                                                           |
| not_contain  | list-value            | 同上                                                                                                                                                                                           |

### 2.4 simulations

simulations中可以包含多个job_group，每个job_group对象必须包含如下对象

| 对象名       | 值类型                | 说明                                   |
| ------------ | --------------------- | -------------------------------------- |
| run_cmd      | list-join_list-scalar | 同builds                               |
| times        | list-join_list-scalar | job_group的执行次数                    |
| seed         | scalar=$$RAND         | 值必须为`$$RAND`，每次执行`$$RAND`都会随机 |
| log          | list-value            | 同builds                               |
| lsf          | scalar                | 同builds                               |
| must_contain | list-value            | 同builds                               |
| can_contain  | list-value            | 同builds                               |
| not_contain  | list-value            | 同builds                               |

### 2.4 demo

```yaml
regr_cfg:
  proj_name: PanGu
  regr_name: DMA  # regress name, recommend use module name
  database_en: True
  database: regress_database.db
  timeout: 10 #0.1  # minute
  max_run_num: 5
  max_process_num: 5
  log_dir: $$ENV_ROOT_DIR/sim
  pre_cmd:
    - make clean
    - cp xx .
  post_cmd: 'bsub merge' 
  lsfs:
    normal: 'bsub -R "rusage[mem=1]"'
    short: 'bsub -q short'
    long: 'bsub -q long'
  parse_cfg: &parse_cfg
    must_contain: 'PASS'
    can_contain: '"error\(s\),lerror,\+error,UVM_ERROR"'  # if have space or specil char should use "", +(,) should use escape code '\'
    not_contain: 'error,fail'

build_common: &build_common
  <<: *parse_cfg
  must_contain: '""'   # modify, null use ""
  lsf: $regr_cfg.lsfs.normal
  wave: 'null'
  ccov: 'off'

sim_common: &sim_common
  <<: *parse_cfg
  lsf: $regr_cfg.lsfs.normal
  wave: 'null'
  ccov: 'off'


builds:
  build0:
    <<: *build_common
    mode: base_fun
    run_cmd:
      -
        - "make anall mode=$mode "
        - "udc='+define+POST_SIM'"
      - "make elab mode=$mode ccov=$ccov wave=$wave"
    log:
      - $mode/log/pds_vhdlan.log
      - $mode/log/pds_vlogan.log
      - $mode/log/elab.cmp_log

simulations:
  tc001_sanity:
    <<: *sim_common
    mode: $builds.build0.mode
    tc: tc001_sanity
    seed: $$RAND
    times: 20
    run_cmd:
      - "make ncrun mode=$mode ccov=$ccov wave=$wave seed=$seed tc=$tc"
    log: $(mode)/log/$(tc)_$(seed).log
  tc002_adm_sanity:
    <<: *sim_common
    mode: $builds.build0.mode
    tc: tc002_adm_sanity
    seed: $$RAND
    times: 10
    run_cmd:
      - "make ncrun mode=$mode ccov=$ccov wave=$wave seed=$seed tc=$tc"
    #log: $(mode)/log/$(tc)_$(seed).log
    log: null
```

## 3 job组成和执行

regress是一个任务提交和管理的工具，任务分为3个层级，task/job_group/job。

### 3.1 job组成

每个pre_cmd/post_cmd/run_cmd可以包含一条或者多条命令，从而可以组成一个或者多个job。
在build和simulation下的job_group中，job除了run_cmd添加的命令，还有`parse_prog + parse_cfg + log`组成的log解析job，有多少个log文件就产生多少个解析job，这些job添加在job_group队列的末尾；不过组这个job的前提是run_cmd中有要执行的命令，如果run_cmd=空/null/None，那么就不会产生解析log的job

- pre_cmd/post_cmd
  所见即所得
  ```yaml
  regr_cfg:
    pre_cmd: "bsub make clean"
    post_cmd: 
      - "bsub make merge"
      - 
        - "cp xx "
        - "../abc -rf"
  ```
  结果如下：
  pre_jobs=["bsub make clean"]
  post_jobs=["bsub make merge", "cp xx ../abc -rf"]

  > 注意：
  pre_cmd/post_cmd的值类型为list-join_list-scalar，二级数组会合并成一个值，所以`post_jobs[1]='cp xx ' + '../abc -rf'`

- job_group.run_cmd
  ```yaml
  builds:
    build0:
      lsf: 'bsub'
      run_cmd: "make elab"
    build1:
      lsf: 'bsub -q short'
      run_cmd:
        - "make anall"
        - 
          - "make elab "
          - "wave=null ccov=null"
  ```
  如果run_cmd为数组
  ```py
  for cmd in job_group.run_cmd:
    job_cmd = job_groups.lsf + ' ' + cmd
  ```

  结果如下：
  build0.jobs=["bsub make elab"]
  build1.jobs=["bsub -q short make anall", "bsub -q short make elab wave=null ccov=null"]

  > 注意：
  run_cmd的值类型为list-join_list-scalar，二级数组会合并成一个值，所以`build1.run_cmd[1]='make elab ' + 'wave=null ccov=null'`

### 3.2 执行顺序

执行顺序的规则如下:
1. pre_cmd/builds/simulations/post_cmd这4个单元是顺序执行的
2. builds和simulations属于task，内部的job_group是并行执行的，并行度由配置regr_cfg.max_run_num控制。
3. job_group内部的多个job是串行执行的
   
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240611110052561-203120937.png)

1. 先串行执行pre_cmd这个job_group内部的所有job
2. 执行完pre_job，然后并行执行builds内部的job_group
3. builds中的所以job_group都执行完以后，执行simulations中的所有job_group
4. simulations中的job_group都执行完以后，执行post_cmd这个job_group

job_group的执行顺序：
虽然job_group的执行是并行的，但是job的提交是串行的，这个job的提交顺序采样广度遍历。
- 广度遍历
如果一个simulation有M个job_group1~job_groupM，每个job_group执行N遍，那么提交的顺序为`job_group1,job_group2...,job_groupM,job_group1,job_group2,...,job_groupM...`
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240611185823873-1506500912.png)
- shuffle
  在regress的命令行中添加`-shuffle`选项，会随机打乱job_group的提交顺序。

异常情况：
1. 在执行pre_cmd/builds/simulations/post_cmd期间，如果用户使用`ctrl+c`退出程序，程序会立刻结束所有任务，更新报告和数据库，然后退出整个回归。
2. 如过pre_cmd执行的任务有报错或异常退出的情况，回归直接结束
3. 执行builds中有job报错或者异常退出，会跳过simulations，直接进入post_cmd

### 3.3 结束状态

每个job执行完都会有一个状态(PASS/FAIL/KILL/TIMEOUT)，job_group根据其所有job的结束状态得到其结束状态
job状态：
- PASS：下面的4种情况都满足才为PASS
  1. job为busb命令，则提交正常
  2. job执行正常结束
  3. job执行没有超时
  4. 如果job为一条log解析命令，则log解析结果为PASS
- FAIL：程序非超时和kill导致的结束，满足下面一个条件则为FAIL
  1. job为busb命令，提交可能异常
  2. job执行异常结束(比如编译报错，仿真报错等，程序结束返回非0值)
  4. 如果job为一条log解析命令，则log解析结果为FAIL
- TIMEOUT：程序正常执行，但是执行时间超过regr_cfg.timeout的时间
- KILL：因为用户ctrl+c导致的job结束，如果该任务为bsub任务，则在退出前会bkill掉，如果是多进程任务，则会kill掉执行的进程再退出。
  
job_group状态：
- PASS：job_group下的所有job的状态都为PASS
- FAIL：job_group下有一个job为FAIL
- TIMEOUT：job_group下有一个job为TIMEOUT
- KILL：job_group下有一个job为KILL

因为job_group下的job是串行执行的，只有前一个job执行结果为PASS/FAIL才会继续执行下一个job，如果job结果为TIMEOUT/KILL那么这个job后面的job不继续执行，整个job_group结束。即job_group结束条件为，所以job都结束了，或者有一个job为TIMEOUT/KILL。

## 4 参数选项

![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240617163234350-621969414.png)

- cfg
  即回归的配置yaml文件
- update_interval
  轮询方式查询各个job状态的间隔时间，这个时间越短job的状态跟新越及时，当然也越浪费系统资源
- db_flush_interval
  数据跟新到数据的间隔时间，时间越短数据库数据跟新越快，db_flush_interval < update_interval没有意义。db_flush_interval设的过小，如果在用户非常多的情况下，可能会出现数据库读写拥塞的问题。
- report_job_cmd
  用于控制log文件的中显示内容，不添加该选项，默认显示log文件，添加则显示job的命令信息
- shuffle
  默认job的执行顺序采用广度优先的遍历方式，使用shuffle则打乱这个顺序
- show_job_info
  默认console只显示jog_group的结束状态信息，添加该选项则会显示job的结束状态信息
- show_start_info
  默认console只显示job_group/job的结束状态信息，添加该信息则会显示job_group/job的开始执行的信息
- debug
  测试模式使用，实际使用中禁止使用