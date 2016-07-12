# PHP项目规范

## 概要
### 目标
更高效的协作开发，沉淀项目组的技术积累。
### 原则
不要重复，无论是你自己还是别人。
### 前提
1. 迁移、重构都是有代价的（不到万不得已不做大规模的重构，可以从写下的下一行代码开始，配合局部的重构，在迭代中完成项目向规范的迁移）
2. 项目不同使用的框架可能不同（大家熟练掌握两三个框架是没问题的）
### 范围
【代码规范】——>【设计规范】——>【类库】——>【框架】
1. 粒度从小到大
2. 强制性从强到弱的

## 代码规范
1. 团队代码风格指南：http://styleguide.baidu.com/style/php/index.html
    * 简单，需要严格遵守；
    * 代码规范保证基本的代码质量、可读性；
    * 命名规范很重要，抛开洁癖、偏见遵守规范就好；
2. FIG代码规范，PHP开源项目协作，发布了一些规范，可以作为参考,http://www.php-fig.org

## 设计规范
这里的设计规范指抛开具体业务逻辑，项目中相对较为较为固定的部分。
1. 统一的参数（http接口、函数）处理：
    * 使用基类或者中间件为你的项目中的所有的control统一的处理$_GET,$_POST,$_REQUEST等参数，禁止直接从这些超变量中获取值；
    * 减少重复冗余的参数检测、校验、转义等代码，避免代码注入等安全问题；

2. 统一且易于使用的日志处理
    * 团队日志数据规范：http://wiki.babel.baidu.com/twiki/bin/view/Com/Inf/LogFormat
    * 日志级别说明：http://wiki.babel.baidu.com/twiki/bin/view/Com/Ecom/Aka/Internal/LoggingStyle
    * 日志接口类示例`Logger::debug()`、`SALoggerModel::getInstance()->debug()`
    * 使用`error_reporting`, `set_error_handler/set_exception_handler`, `error_log`, `display_errors`, `trigger_error`等函数和选项确保错误都能得到及时的反馈和记录。

3. 统一的错误反馈机制
    * php中几种错误反馈形式：`return` , `throw exception`（推荐）, `trigger_error`；
    * 在代码中统一的使用其中一种，封装以一些便利函数来简化代码，举个例子：
    ```php
    function foo() {
        // ...
        // ...
        if ($ret['status'] == 'error') {
            return array(
                'result' => false,
                'message' => 'xxx error',
                );
        } else {
            return array(
                'result' => true,
                'message' => 'xxx error'
                );
        }
    }

    function foo() {
        // ...
        // ...
        if ($ret['status'] == 'error') {
            throw new Exception("xxx error", XXX_ERROR);
        }
        return $result;
    }
    ```
    * 尽量早的以中断的方式抛出错误（`return/throw`)，避免过于复杂的错误处理逻辑；

## 类库
类库的沉淀与积累是减少工作量的有效手段。  
1. 通用，业务无关：
    * http；
    curl库用起来比较繁琐，记忆力不好的人经常要去翻手册，Requests(https://github.com/rmccue/Requests)

    * logger
    没有合适的logger库，另外团队有自己的日志规

    * ORM
    ORM优势，抽象、简化数据库操作，有一些限制，不能支持复杂的查询，但是同时这些限制能够帮助我们做出正确的限制，在领域中识别出本质的对象及它们之间的关系。
    常用的模式有Active Record、Data Mapper，前者简单粗暴，后则解耦做的更好，在domain model和数据库中加入了mapper层，可以在这一层中事务、缓存等特性。RedBean(http://redbeanphp.com/)

    * config
    支持production、development、testing三个环境配置，支持多配置文件

    * template engine
    大同小异，Smarty(http://www.smarty.net/)

2. 团队业务相关：
    pss\bcs\pcs\OpenApi\OAuth\Rig

3. 如何沉淀我们的类库？
    * 申请SCM模块来放可以共用的库；
    * 一个库对应一个子模块；
    * 一个库一个文件夹（即使只有一个文件）；
    * 库中不使用autoload，库中文件使用绝对地址进行加载，用户只需require一个文件；
    * 支持PHP 5.2.10+以上版本；

## 框架
1. odp（http://odp.baidu.com/)
    * 与团队社区类业务契合度高；
    * 集成了各种框架、模块、工具；
    * 比较重，online安装200M+；
    * PHP 5.2.17；
    * 有些库的风格比较老，比如RAL，比curl好不了多少；
2. codeigniter（http://codeigniter.org.cn/)
    * 简单，小巧，文档丰富，可扩展性好；
    * 集成了基本的工具类库，第三方扩展少；
    * 适用于php 5.2+，bae（内外网）；
3. Laravel（http://www.golaravel.com/)
    * 抛弃了PHP 5.2的一些历史包袱，设计优秀；
    * PHP 5.3.17+，依赖Composer，第三方扩展多；
    * 内置ORM、模板引擎；

## 其它

### 项目主页
     * 项目词典，中英文，文档、代码中命名的重要参考
     * 基本抽象、对象以及它们之间的关系
     * 坑（举个栗子，影票）
     * 好用的工具
     * 优秀的文档、书籍
     * 分享
### 项目管理，代码管理，上线流程
     * icafe、scm、SVN教程
     * BAE项目版本管理与上线规范

【参考】
* [awesome-php](https://github.com/ziadoz/awesome-php)
* [PHP的PSR规范中文版](http://feiyang.me/2013/03/php-psr-in-chinese/)
* [程序接口设计、错误处理与应用日志](程序接口设计、错误处理与应用日志.pdf)