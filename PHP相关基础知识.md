# PHP相关基础知识
## 变量
分为基本类型（数值，字符串，布尔值，数组）和引用类型（对象；
变量根据其指向的值在传递的时候分为值传递和引用传递；
数组变量是值传递（深拷贝）；

## 错误处理
1. error_reporting控制错误报告的级别，当设置为0时所以的错误信息都会被屏蔽；
2. set_error_handler设置用户自定义的错误处理函数，用户函数会忽略error_reporting的设置而被调用；
3. display_error控制错误信息的输出以及输出的地址，会受到error_reporting的影响；
4. error_log选项指定php错误文件，受到error_reporting的影响；
5. set_error_handler只能捕获warning、notice、restrict和用户触发的错误trigger_error；
6. parse、core等错误只会输出到日志文件，不会通过stdin输出；
7. error-control operator（@）会disable error reporting for critical error, but the script will die right there without any indicate.；
8. error_log函数输出信息到错误日志文件；
9. trigger_error触发用户自定义错误，被error handler捕获；
10. 自定义的error handler需要根据error_reporting的设置来处理错误；
11. 默认的错误信息容易淹没在html代码中，需要通过显示页面源码查看；
