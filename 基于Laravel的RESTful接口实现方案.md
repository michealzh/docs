# RESTful接口实现方案
本项目结合业务实际情况和以往经验，在Laravel/Lumen框架的基础上，制定了一个定义清晰、表达灵活、易于实现的RESTful API实现方案。

## RESTful介绍
REST的全称是【资源表征状态转移】，又可以拆解为资源、表征、状态转移三个部分。
* 资源表征：如何定位定位互联网上的一个资源？使用URI/URL，http://${domain}/app/1/page2
* 表征状态：如何表征这个资源？使用JSON、XML、二进制等等，只是资源的一个视图。
* 状态转移：如何对资源进行操作，资源的状态如何转移？使用HTTP method来表达对资源CRUD操作，资源在服务器端发生状态转换。

RESTful的一大难点在于资源查询语义的实现，有基于http querystring或者body两种方案，使用querystring的好处是方便但是表达能力较弱，并且没有成熟的方案；使用body的好处是灵活并且表达能力强，![graphQL](https://facebook.github.io/react/blog/2015/05/01/graphql-introduction.html)是facebook提供的一种较为成熟的方案。

## RESTful-like API接口规范
本规范并不严格遵守RESTful风格的指导，比如考虑到兼容性情况并不使用GET、POST之外的其它HTTP动词。

### 资源模型
说明：
1. 各模型的id字段由系统自动生成
2. 各模型的created_at、updated_at、deleted_at由系统自动生成和维护
3. 字段约束：类型(`string`,`integer`,`array`)，字符长度限制（`max:${num}`）,枚举值(`in:${value1},${value2}`)

例子：
```
{
    "id": 123456, // baidu uid, integer
    "name": "张三", // patient name, string|max:50
    "sex": "male", // string|in:male,female
    "birthday": "2015-11-22", // YYYY-MM-DD, string|max
    "occupation": "学生", // string|max:50
    "address": "北京市海淀区上地东路1号", // string|max:200
    "phone": "13812341234", // string|max:20
    "created_at": "2015-08-21 18:20:49", // create time, string|max:30
    "updated_at": "2015-08-21 18:20:49" // modify time, string|max:30
}
```

### 模型定义方法
采用传统的JSON schema的方式对模型定义较为繁琐，并且只能对属性的类型和值进行约束，无法表达模型之间的关系。(Laravel Validation模块)[http://www.golaravel.com/laravel/docs/5.0/validation/]使用了一种较为简便的规则表述方法，各种规则以`|`分隔，规则名与规则值之间以`:`分隔。本模型定义方法在Laravel Validation模块已有的基础上进行扩展，是Laravel Validation模块的超集。

#### Laravel Validation规则
* Accepted
* Active URL
* After (Date)
* Alpha
* Alpha Dash
* Alpha Numeric
* Array
* Before (Date)
* Between
* Boolean
* Confirmed
* Date
* Date Format
* Different
* Digits
* Digits Between
* E-Mail
* Exists (Database)
* Image (File)
* In
* Integer
* IP Address
* JSON
* Max
* MIME Types (File)
* Min
* Not In
* Numeric
* Regular Expression
* Required
* Required If
* Required Unless
* Required With
* Required With All
* Required Without
* Required Without All
* Same
* Size
* String
* Timezone
* Unique (Database)
* URL

#### 扩展规则
* default: 默认值
* refer: 引用
* belongs: 从属
* auth: 鉴权字段
* array_str: 逗号分隔的数组形式
* array_in: array_str字段in条件

### 资源操作
基于JSON表征的RESTful接口风格来实现对系统中资源模型（文件除外）的列举、获取、添加、修改、删除操作。接口描述中以`${varibale}`的方式表达变量。
基本请求格式如下：
```
(GET|POST) /api/${resource}/{$method} HTTP/1.1
content-type: application/json
...
${JSON-base request body}
```
操作成功返回：
```
HTTP/1.1 200 OK
content-type: application/json
...
${JSON-base response body}
```
操作失败返回：
```
HTTP/1.1 500 Internal Error
content-type: application/json
...
{
    error_code: ${error code},
    error_msg: ${error msg}
}
```

### 查询和列举（query）
格式：
`GET /api/${resource}/query?${querystring}`
参数：
* `_skip=${n}`: 跳过n跳记录，默认为0，可选
* `_limit=${n}`: 限制返回条数，默认为20，[0,100)之间，可选
* `_asc=${attr}`: 按照字段正序返回，可选
* `_desc=${attr}`: 按照字段逆序返回，可选
* **`_wd=${keywords}`: 关键字搜索**
* `${attr}=${value}`: 按照字段与值相等为条件过滤列表，可选，可添加多条，一个字段不允许出现多次
* `${attr}=in:${value1},${value2}`: in查询条件
* `_pull=${attr1},${attr2}`: 拉取字段
* `_orderby=${key1}:[asc|desc],${key2}:[asc|desc]`: 多字段排序

返回：
```
{
    total: ${total result num},
    count: ${result num},
    items: [
        {
            id: ${resourceId},
            ${attr}: ${value},
            ...
        },
        ...
    ]
```
### 获取（info）
格式：
`GET /api/${resource}/info?id=${resourceId}`
参数：
`id=${resourceId}`: 资源ID，必填
返回：
```
{
    id: ${resourceId},
    ${attr}: ${value},
    ...
}
```
### 创建（create）
格式：
```
POST /api/${resource}/create
{
    ${attr}: ${value},
    ...
}
```
参数：
`id=${resourceId}`: 资源ID，必填
返回：
```
{
    id: ${resourceId},
    ${attr}: ${value},
    ...
}
```
### 更改（update）
格式：
```
POST /api/${resource}/{$resourceId}/update
{
    ${attr}: ${value},
    ...
}
```
参数：
`id=${resourceId}`: 资源ID，必填
返回：
```
{
    ${attr}: ${value},
    ...
}
```
### 删除（remove）
格式：
`POST /api/${resource}/{$resourceId}/remove`

### 错误码
待补充。