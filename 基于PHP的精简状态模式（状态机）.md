# 基于PHP的精简状态模式（状态机）

状态模式可以有效的管理对象的状态以及状态之间的转换。

传统的状态模式实现较为复杂，需要建立context和state对象以及之间的相互调用。具体参见：http://design-patterns.readthedocs.org/zh_CN/latest/behavioral_patterns/state.html

下面用代码介绍之一种基于PHP的精简实现方式，省去了context对象，创建和使用都比较灵活。主要看注释：
```
<?php

/**
 * 状态抽象类，主要实现操作的默认
 */
class Media_State {

    // 状态的默认字符串表征
    public $status = 'init';

    /**
     * 状态操作列表，新增的操作都需要在抽象类添加，然后在
     * 目标状态类中实现操作，并发回转换后的状态
     */

    public function report() {
        return $this;   // 默认返回状态对象本身，即不会有状态改变
        // throw new Exception("not-allowed", 1); // 可选，不允许的操作发生时及时报错
    }

    public function pass() {
        return $this;
    }

    public function fail() {
        return $this;
    }

    public function off() {
        return $this;
    }

    public function on() {
        return $this;
    }

    // 状态工厂类，通过字符串得到相应的状态实例
    public static function get($status) {
        $clazz = ucfirst($status).'MediaState'; // demo: when $statue = 'init'，$clazz = InitMediaState;
        return new $clazz($status); // demo: return new InitMediaState($status);
    }

    // 字符串转换函数，方便将状态对象直接当做字符串使用
    public function __toString() {
        return $this->status;
    }
}

/**
 * 具体的状态，需要实现允许的操作以及返回操作后的状态对象
 */
class InitMediaState extends Media_State {

    public $status = 'init';

    public function report() {
        return new ReportedMediaState();
    }

    public function pass() {
        return new OnlineMediaState();
    }

    public function fail() {
        return new ForbiddenMediaState();
    }

}

class ReportedMediaState extends Media_State {

    public $status = 'reported';

    public function pass() {
        return new OnlineMediaState();
    }

    public function fail() {
        return new ForbiddenMediaState();
    }
}

class PendingMediaState extends Media_State {

    public $status = 'pending';

    public function pass() {
        return new OnlineMediaState();
    }

    public function fail() {
        return new ForbiddenMediaState();
    }
}

class ForbiddenMediaState extends Media_State {

    public $status = 'forbidden';

    public function pass() {
        return new OnlineMediaState();
    }
}

class OnlineMediaState extends Media_State {

    public $status = 'online';

    public function off() {
        return new OfflineMediaState();
    }
}

class OfflineMediaState extends Media_State {

    public $status = 'offline';

    public function on() {
        return new OnlineMediaState();
    }
}

/**
 * 使用例子
 */
// 从数据库中（或者其它地方）获取记录
$record = Utils_Db::findOne($id);
// 从状态字符串得到状态对象：reported
$state = Media_State::get($record['status']);
// 审核不通过，执行fail操作，并接住返回的新状态
$state = $state->fail();
echo 'status: '.$state; // status: forbidden
```