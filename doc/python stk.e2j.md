# e2j(excel2json)

读取reg/itf/crg类型的excel文件，生成对应的json文件

## 1. 使用

- 前提：
  已经有编写好的reg/itf/crg的excel文件，具体格式参考stk.genExcel
- 命令:
```sh
stk e2j --mode [reg/itf/crg] xxx.xlsx
```
- 输出：默认在当前目录下会创建一个jsons目录，存放对应的json文件。--dri可以设置存放json文件目录的名字

## 2 其他
