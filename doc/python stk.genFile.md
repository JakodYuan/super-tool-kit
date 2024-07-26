# genFile

使用json中的配置，和jinja2语法编写的模板文件生成文件。

## 1. 使用

- 前提：
    有编写好的template文件，和与template相匹配的json配置文件（main/common）
- 命令：
    - 以main_cfg.json为主配置，common_cfg.json为通用配置，if.jinja为模板文件进行渲染，生成的文件后缀为.sv
    `python genFile.py -m main_cfg.json -c common_cfg.json -t if.jinja -e .sv`
 
    - jsons中的所以json文件都为主配置，common.json为通用配置，agent下所以jinja文件为模板进行渲染，输出文件放在/out/env/agent目录下，后缀名都为sv。有m个主json文件，n个模板文件，则会生成m\*n文件，内部渲染m\*n次，每次渲染取3个文件 main_cfg.json(1个)+common.json+template.jinja(1个)
    `python genFile.py -m ./jsons/*.json -c ./common.json -t ./templates_super/env/agent/*.jinja -d ./out/env/agents -e .sv`
 
    - jsons中的所以json文件都为主配置，common.json为通用配置，tc.jinja文件为模板进行渲染，输出文件放在/out/目录下，后缀名都为sv。将所有主配置合并为一个主配置，有m个主json文件，n个模板文件则会生成n文件，内部渲染n次，每次渲染取m+2个文件 main_cfg.json(m个)+common.json+template.jinja(1个)
    `python genFile.py -m ./jsons/*.json -c ./common.json -t ./templates_super/tc/tc.jinja -g ifs -d ./out/tc -o tc001_sanity -e .sv`
 
    - 没有主配置，只有common配置，以tcs.jinja为模板，生成1个文件，方在out/tc目录下，文件为tcs.sv
    `python genFile.py -m ./common.json -t ./templates_super/tc/tcs.jinja -d ./out/tc -e .sv -o tcs`
 
    - 没有配置，以svk_pkg.jinja为模板，生成1个文件，方在out/env目录下，文件为svk_pkg.sv这种没有配置的渲染，其实就是简单的文件拷贝，然后改了后缀名
    `python genFile.py -t ./templates_super/env/svk_pkg.jinja -d ./out/env -e .sv`

## 2. 选项

- **-m**：(main)
    主配置json文件，一个主json文件+一个template会生成一个文件；可以使用"\*"进行通配，-m选项可以使用多次。`-m abc.json` or `-m abc.json -m xyz.json` or `-m ./jsons/\*.json`
- **-c**：(common)
    共享json文件，包含所以文件渲染都需要的配置。，可以使用"\*"进行通配，-c选项可以使用多次。`-c common.json` or `-m a.json -b.json` or `-m common/\*.json`
    ![img](https://img2023.cnblogs.com/blog/898240/202311/898240-20231102141758312-669274064.png)
- **-t**：(template)
    模板文件，jinja语法编写的模板文件，，可以使用"\*"进行通配，-t选项可以使用多次。`-t env.json` or  `-t env.json -t env_cfg.json` or `-m ./env/\*.json`
- **-p**：(password)
    模板有加密时，输入密码
- **-e**：(extension)
    生成文件后缀。`-e .sv`
- **-o**：(output_name)
    输出文件名，只有在只生成一个文件时有效。默认为`{prefix}{main_cfg.name}_{temp_name}`。如果-m或者-t选项使用了"\*"通配，会生成多个文件，比如-m匹配了2个json文件，-t匹配了3个模板文件，那么会生成6个文件，
- **-d**：(directory)
    设置输出文件目录，默认为'./out'
- **-g**：(merge)
    `-g abc`，在cfg中添加一个key为"abc"的成员，成员值为list，即：`cfg['abc'] = []`的成员，且把main json的值添加到list中。例如:`-g ifs`,-m匹配了abc.json和xyz.json，cfg添加`cfg['ifs'] = [abc, xyz]`
- **-i**：(include path/dir)
    设置模板搜索路径，程序回去设置的路径下去搜索需要渲染的模板文件，默认会添加当前路径。-i可以使用多次。(import/include/get_template())
- **-diff**：
    开启diff_merge功能，如果之前生成的代码已经进行了部分修改，但是又想通过genFile生成新内容，此时可以开启该功能，实现手写代码与新生成代码的合并，类似于svn/git/p4代码冲突的合并操作。

    开启合并功能后，可设置合并模式:
    - **--add**：
        模板内容相对原有文件有新增行的时候，可以选择 new:添加新增行， com_new:添加新增行，并注释
    - **--remove**：
        模板内容相对原有文件有删除行的时候，可以选择 old:保留删除行， com_old:保留删除行，并注释
    - **--change**：
        模板内容相对原有文件有改变行的时候，可以选择 old:使用原有行， new:使用新行，com_old:使用新行，注释原有行，com_new:使用原有行，注释新行
    - **-comment_flag**：
        行注释标记，默认为"//"

## 3. 模板与json

渲染流程
1. 会先读取json文件，生成一个名字为cfg的python对象
2. 使用cfg对象来渲染模板文件

模板文件中可以调用cfg对象，渲染的时候会替换成cfg对象实际的值，问题在于cfg中会有什么值。
1. 在没有使用-g选项时，cfg为main.json和common.json的合并的结果，比如
main.json
```json
{
    "addr": "0xfff0",
    "data": "0x8989"
}
```
common.json
```json
{
    "author": "Jakod",
    "email": "JakodYuan@outlook.com"
}
```
`-m main.json -c common.json`：得到的
```py
cfg={"addr":"0xfff0",
     "data":"0x8989",
     "author":"Jakod",
     "email":"JakodYuan@outlook.com"}
```

2. 使用了-g xx选项，cfg字典包含一个xx对象和common.json中的成员，其中xx对象为列表，包含所以匹配到了main.json

main1.json
```json
{
    "addr": "0xfff0",
    "data": "0x8989"
}
```
main1.json
```json
{
    "addr": "0x6666",
    "data": "0x5afa"
}
```
common.json
```json
{
    "author": "Jakod",
    "email": "JakodYuan@outlook.com"
}
```
`-g mains -m main*.json -c common.json`：得到的
```py
cfg={"mains":
        [{"addr":"0xfff0", "data":"0x8989"},
         {"addr":"0x6666", "data":"0x5afa"}], 
    "author":"Jakod", 
    "email":"JakodYuan@outlook.com"}
```

## 4. cfg中的特殊对象

1. cfg['time'] = time对象
2. cfg['bs'] = buildins对象，包含一系列类型转换的对象
3. cfg['os'] = os对象
4. cfg['file_name'] = 生成文件名