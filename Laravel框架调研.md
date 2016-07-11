
# Laravel调研
## 调研方向
* laravel框架功能优势
* laravel框架性能情况
* 与业务的兼容性
## 功能优势
### 模块化
#### composer包管理器
* 借鉴NPM
* PEAR非常难用
#### PHP-Fig
* 规范：style guide, autoloading, interfaces(logger, message)
* 成员：composer, laravel, Yii, Zend Framework 2 etc.
### 整体设计
* 更合理的routes, middleware, controller(ap的路由围绕module/controller/action, 坑多)
* 更好的controller、models、view支持
### 解耦合
* 依赖注入、服务容器等
### 文档、社区
http://packalyst.com/

## 性能
ab在100并发下大概比ap（yaf）慢一倍，系统负载差别不大：http://www.ttlsa.com/php/yii-yaf-ci-php/
* 'hello world!' 请求，代码时间,laravel:20ms, ap:0.5ms, 实测
* 在以业务逻辑为主、偏网络交互的系统中框架的性能不是瓶颈
```
target  并发/次数   time
laravel 20/1000    449.799
ap      20/1000    23.235
laravel 1/1000     22.916
ap      1/1000     1.216
```
报表：http://yanlong.fe.baidu.com/laravel/profiling.txt

## 兼容性
ODP/ORP并没有强依赖于AP框架，RAL、Bd_Conf、Bd_Log、Bd_Db_ConnMgr等服务或类库都可以通过在laravel框架中使用。
```
/**
 * @file Bd_Init.php ODP初始化入口
 */
    public static function init($app_name = null)
    {
        if(self::$isInit)
        {
            return false;
        }
        self::$isInit = true;
        // 设置默认timezone，避免PHP5.4或HHVM报warning
        date_default_timezone_set('PRC');
        // 初始化基础环境
        self::initBasicEnv();
        // 初始化App环境
        self::initAppEnv($app_name);
        // 初始化Ap框架
        self::initAp();
        // 初始化日志库
        self::initLog($app_name);
        // 执行产品线的auto_prepend
        self::doProductPrepend();
        return Ap_Application::app();
    }
```
替换AP框架:
```
/**
 * webroot/$app/index.php
 */
<?php
// Bd_Init::init()->bootstrap()->run(); // 初始化ODP，返回AP框架的app实例，run
Bd_Init::init();    // 只初始化ODP
require_once ROOT_PATH.'/../orc/laravel/public/index.php';  // 转向laravel入口
```
在ORP环境中使用laravel：
```
    /**
     * Display the specified resource.
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        // 使用laravel/ORM访问数据库
        $p = new Photo;
        $p->name = 'fff';
        $p->save();
        $results['from laravel/orm'] = Photo::all();
        error_reporting(0);
        // 使用ODP/ral访问数据库
        require_once APP_PATH.'/mediaspot/library/utils/Db.php';
        $results['from odp/ral'] = \Utils_Db::find('treatment');
        echo json_encode($results);
    }
```

## 结论：
1. laravel的价值在路由、中间件、MVC以及系统本身的架构，对团队的效率和技术积累都有好处
2. 性能不如ap，对于非高并发情况下对页面速度影响可以忽略，大流量情况下对机器资源消耗相对较高
3. 在合理设置autoload的情况下，odp中的其它组件可以比较方便的应用到laravel框架中来；在Ap中积累的一些代码需要进行namespace和模块导入相关的重构；
4. 后续orp lightapp/lightapp3会由RD自运维，op应该不会阻挠我们做试验；
5. 需要找个项目踩踩坑，积累一下经验和规范

