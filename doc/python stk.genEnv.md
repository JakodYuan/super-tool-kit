# genEnv

根据excel的接口列表生成验证环境，接口列表的填写和格式参考genExcel

## 1. 使用

命令
1. `stk genEnv xxx.excel --common common.json`：生成基于UVM的验证环境
2. `stk genEnv -s xxx.excel --common common.json`：生成基于SVK的验证环境
将会在当前目录下生成一个out文件夹，文件夹内为生成的环境，验证环境的文件结构如下
![img](https://img2023.cnblogs.com/blog/898240/202407/898240-20240725175047729-704744078.png)

- cfg：放filelist等配置文件
- env：放env、agent、rm等
- sim：仿真目录，包含一个sourceme文件
- tc：测试用例目录
- th：验证环境的接口连接和harness相关文件

common.json需要包含如下内容：
- author：用户名，用于显示在生成文件的文件头中
- email：邮箱地址，用于显示在生成文件的文件头中
- prj：项目名，用于设置生成文件的名字，比env的名字
- top：要测试模块的module名字，用于生成module例化
- i_top：module的例化名

```json
{
  "author": "Jakod",
  "email": "JakodYuan@outlook.com",
  "prj": "kirin",
  "top": "gpu",
  "i_top": "u_gpu"
}
```

## 2. UVM与SVK

其实基于SVK的验证环境也是最终也是基于UVM，只是在UVM的基础上添加了一些扩展。传统的UVM验证环境结构分为如下2大部分：
1. harness.sv中例化interface，并将interface与DUT连接起来，同时将interface，config_db到验证env或者agent中。
2. env.sv中例化agent，同在agent中将sqr与driver连接起来，driver接收vertual interface并驱动。

如果新增接口，需要修改harness.sv和env.sv，涉及的文件比较多，不符合软件的开闭原则。
基于SVK的验证环境通过静态注册+UVM factory模式，实现了agent自动例化和配置，只需要添加一个bind文件，interface连接+agent例化的工作，不需要修改harness.sv和env.sv，完美实现了软件对扩展开放，对修改封闭的原则。

实现方式：
1. bind文件中例化interface并根据模式进行连接
2. bind文件中包含一个bind_core的类，类中包含了agent的信息，该类进行静态例化，并把句柄添加到abs[$]队列中
3. svk_env中遍历abs[$]队列，根据bind_core的信息，例化agent并驱动

这种方式把一个agent涉及的内容集中到了bind文件中，修改或者新增都很方便，符合提取变化点的需求。

## 3. 环境的使用与修改

生成的验证环境可以通过stk.bsim直接运行
进入：sim目录，执行`stk bsim sim`，即可进行编译和仿真

由于agent还是空壳，例化的module是假的，所以达到真实的测试环境，还需要对环境进行修改。

1. 在cfg/tb.f中添加RTL的filelist，且要放在harness.sv前面，当然RTL的filelist也可以独立成一个filelist
2. 注释掉cfg/tb.f中的top.sv，这是一个假的被测module，用RTL中的真实module替代
3. 完善agent中的driver
4. 解注释interface的链接，如果使用的是基于UVM的环境，这部分内容在/th/harness.sv中。如果使用的是基于SVK的验证环境，这部分内容在/th/xx_bind.sv文件中。
5. 在sim目录下执行`stk bsim sim`即可完成编译和仿真。