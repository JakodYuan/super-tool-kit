# run

使用yaml存放一些常用的简单操作，将这些操作转换为一个特定的命令，方便执行。

## 1. 使用

以p4上传为例，通常需要先reconcile，然后submit，这两个操作的命令都添加到cmd.yaml中，转换为一个up的命令

cmd.yaml
```yaml
up:
    cmd:
        - p4 reconcile ...
        - p4 submit -d "update"
```

可以直接执行`stk run up`，会指向cmd中的两条命令

