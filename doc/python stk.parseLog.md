# parseLog


## 使用说明

parseLog用来解析log文件，得出编译和仿真PASS/FAIL的结论，解析根据3类关键词句来处理：

- must_contain: 
    log必须匹配该关键词句才能判定为PASS
    区分大小写
- can_contain：
    log的某一行匹配上该关键词句不会判定为FAIL，尽管该行匹配上了not_contain。该关键词句用来排除not_contain中可以包含的关键词句，比如not_contain=Error，但是log中出现'is not an Error'也会认定为错误，设置can_contain='is not an Error'则可解决该问题
    有一种特殊情况，一行中有多个错误信息，一个可以匹配can_contain一个不能匹配con_contain，依然会判定为FAIL。比如UVM_ERROR is not an Error，这里UVM_ERROR不在can_contain中，所以会判定为FAIL。
    区分大小写
- not_contain:
    log中某一行出现该关键词句会判断为FAIL，除非该行又匹配到了can_contain关键词句
    不区分大小写
> 注意：
1. 多个关键词句通过','分隔，如果关键词句中包含','用'\,'来代替
2. 如果关键中有空格或者`(,)`等特殊字符，那么整个参数需要用单引号''或者双引号""来包裹，推荐无论关键词句是否有空格，整个参数值都用引号包裹`parseLog --can_contain 'is not an Error'`
3. 关键词句中包含了特殊字符，比如：`+、*、(、)、[、]`等正则表达式的特殊符合，需要对其添加'\'进行转义

## 特性

1. 逆向解析文件，提高解析效率，因为log如果报错都会出现在末尾，逆向解析可以提前得出解析结果，结束解析。
2. FAIL时，输出匹配到的错误信息，或者输出需要匹配而未匹配的关键词句
3. PASS时，输出匹配到需要匹配的信息

输出信息
- PASS
    ```sv
    ...   // 匹配到的must_contain信息
    PASS
    ```

- FAIL
    ```sv
    ...   // 匹配的not_contain信息，或者提示某个must_contain的关键词句未被匹配到
    FAIL
    ```
