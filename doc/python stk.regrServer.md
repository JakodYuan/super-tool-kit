# stk.regrServer

为了方便回归信息的查看，管理和统计，regrServer工具以网页表格，图表等形式，非常直观的展示回归数据，方便项目管理。

[TOC]

## 1 使用说明

### 1.1 参数说明与使用

| 参数选项 | 说明 |
|---|----|
| --host          | regrServer的host ip号，使用默认值即可|
| --port          | regrServer的端口号|
| --regress_db    | 回归数据存放的数据库，如果不存在会创建|
| --user_db       | 用户数据存放的数据库，如果不存在会创建|
| -view_need_login| 查看回归数据是否需要登录，默认不登录也可以查看|
| -use_admin      | 是否使用隐藏的admin账号，该账号推荐只有第一次设管理员的时候使用 |
| -use_matplotlib | 默认情况下使用javascript，然而如果浏览器比较老旧，特别是linux的firefox，则不支持该功能，此时必须添加use_matplotlib选项用matplotlib来显示图表 |

1. 推荐使用命令`stk regrServer --regress_db xxx.db --user_db xxx.db`
2. 在浏览器中输入：localhost:8888，如果是其他机器上访问，则根据程序的输出地址来访问，如下面的：127.18.133.6:8888
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240612190349092-224269616.png)
3. 当然如何觉得IP难记和输入，可以修改/etc/hosts,添加`127.18.133.6 stk`，同时刷新DNS或者重启网络，在浏览器输入`stk:8888`即可访问。
```sh
# 刷新DNS
ipconfig /flushdns                      # windows
sudo systemctl status NetworkManager    # 查看NeworkManager状态
sudo systemctl start NetworkManager     # 启动NM
sudo service networking restart         # 重启NM
```


### 1.2 数据页面说明

stk.regrServer将stk.regress回归产生的数据以网页的形式展现出来。网页分为4个层级。
| 层级 | 说明 |
|---|---|
|index | 列出所有项目的汇总信息，包括总回归数，通过数等，还可以点击graph，查看分时段的统计数据。|
|proj| 列出某个项目下，所有用户的回归信息，也可以通过点击graph，查看每个用户的分时段统计数据|
|user| 列出某个用户的所以回归的汇总信息，点击某个回归可以查看回归的具体信息|
|regr| 列出回归的任务的具体信息，这些任务的排列顺序为TIMEOUT->FAIL->KILL->PASS，即错误用例会显示在最前面，不同状态标记为不同的颜色|
|log| 网页显示log内容，并对包含error和fail的行进行高亮，linux中还会通过gvim打开log文件|


### 1.3 管理页面说明

regrServer除了展示数据，还有用户管理和数据删除功能，该功能包括了注册页面，登录页面，和后台管理页面。
1. 用户管理：用户查看数据需要注册账号，用户管理功能则用来审批注册申请，和用户权限管理。用户权限分为5级
    - 0：注册成功后的默认权限，无法登录
    - 1：可以登录账号
    - 2：可以登录，且可以删除用户个人回归数据
    - 3：可以登录，且可以删除所有用户数据
    - 4：在3的基础上，可以进入后台页面，管理所有用户的权限、删除用户和批量回归数据删除。
2. 数据删除：某个回归有问题，删除该回归数据，或者数据时间太久远，没有价值即可删除。删除数据有权限要求，这个权限设置就依赖于用户管理功能。

## 2 页面展示

每个页面左上侧有一个导航，右上侧有一个登录/退出链接，如果登录的账号权限为4，那么左上方还会显示后台链接。未登录时右上方显示login链接，登录成功后显示"Welcome, xxx!",点击会显示logout的下拉菜单。

### 2.1 index

![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240612190548954-1915727953.png)
登录的是账号权限为4的时候，表格的右侧有删除按钮，可以删除整个项目的数据。
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240612190902498-1924192383.png)

### 2.2 proj

登录账户账号的权限等于4时，在表格的末尾显示delete按钮，删除整个用户的数据
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240612191230190-226903347.png)

### 2.3 user

登录账户名字与数据用户名相同，且权限=2时，或者账号的权限大于等于3时，在表格的末尾显示delete按钮，删除整个回归的数据
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240613101944062-461027936.png)
- regr_name：回归名=yaml.regr_cfg.regr_name+datetime
- build/case/p：分别显示:总jobgroup数(build数量，testcase数量，执行并行度)
- status：回归的状态，分别有：BUILD/RUN/FINISH
- run_time：回归的时间
- pass：分别在build/run阶段显示pass的jobgroup数量
- fail：同上
- finish：同上
- run：同上
- idle：同上
- all：同上
- pass_ratio: 同上
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240626103320698-791716376.png)

### 2.4 regr

点击user页面的"regr_name"页面会调整到regr页面，该显示每天用例的状态、命令，log等相关信息，排列顺序为TIMEOUT->FAIL->KILL->PASS,且用不同颜色表示。

![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240612191714520-564687964.png)
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240614181937194-1574697051.png)
如果用户登录了，且查看回归的user_name与登录的用户名相同，那么在页面顶部会有一个"open log with gvim"的单选框，选中后点击log链接会通过gvim的方式打开log文件(目的是gvim都个性化高亮了错误内容，且方便在log中跳转和查找)。如果没有登录或者没有选中单选框，点击log链接会跳转到log页面显示log。

### 2.5 testcase

点击user页面的"build/case/p"列，会跳转到testcase页面，该页面为build和testcase的列表，点击"+"会显示各个build/testcase的详细信息，再次点击"-"，会折叠该信息。
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240626143641255-1229411956.png)

与regr页面一样，如果用户登录了，且查看回归的user_name与登录的用户名相同，那么在页面顶部会有一个"open log with gvim"的选项。

### 2.5 log

显示log内容，并对包含error和fail的行进行高亮
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240613151430246-1545387158.png)

### 2.6 register

![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240612191934341-154453159.png)

### 2.7 login

![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240612191850601-1676950467.png)

### 2.8 admin

![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240613100451485-93486684.png)

### 2.9 graph


如果regrServer没有使用`-use_matplotlib`选项且浏览器支持javascript的新特性，在index和proj页面点击show会悬浮显示图表
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240626102240282-1835636300.png)
如果浏览器不支持javascript新特性，则为如下显示：
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240626102430880-408129126.png)
即无法显示图表，此时有两个选择：
1. 启动服务器的时候使用`-use_matplotlib`选项，不使用javascript来显示图表，点击show则会显示如下页面
![img](https://img2023.cnblogs.com/blog/898240/202406/898240-20240613101012089-959123467.png)
2. 更新浏览器，以firefox为例，步骤如下：
   1. 下载最新firefox:https://www.mozilla.org/en-US/firefox/new/
   2. 解压：`tar xjf firefox-*.tar.bz2`
   3. 移动解压文件到opt目录`sudo mv firefox /opt/`
   4. 添加链接到bin目录下，以便可以直接调用firefox：`sudo ln -s /opt/firefox/firefox /usr/bin/firefox`
   5. 启动新的firefox:`firefox`