# initcalls and modules



## 1 How to define a module

从GPT上查到设置一个模块的方法如下：

![img](./.09_initcall_module/lu3223621jpo0c_tmp_cc7f132801f27dcf.png)

![img](./.09_initcall_module/lu3223621jpo0c_tmp_96ed502060a5ae73.png)

![img](./.09_initcall_module/lu3223621jpo0c_tmp_6b186b9366a8e2ea.png)



## 2 The mechanism behind

### 2.1 Source code checking

![img](./.09_initcall_module/lu3223621jpo0c_tmp_7707bd561bf82f58.png)

->

![img](./.09_initcall_module/lu3223621jpo0c_tmp_90fdf91c6c1d2f5d.png)

->

![img](./.09_initcall_module/lu3223621jpo0c_tmp_7af0a6cb3e60e43b.png)

->

![img](./.09_initcall_module/lu3223621jpo0c_tmp_7d930d353c5c02c8.png)

->

![img](./.09_initcall_module/lu3223621jpo0c_tmp_2135106440d78bf3.png)

如果没有定义MODULE，即，相应的驱动会被编译到内核中。那么，宏module_init(x)会使相应的模块初始化函数被映射到.init段。

如果要编译为module，而不是builtin到内核中，就需要定义MODULE（需要确认是不是这样）。
