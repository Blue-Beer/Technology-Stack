# RESTful接口的后端实现

实现接口的背景是清晰架构 [clear-architecture](../clear-architecture/clear-architecture.md) 的后端项目结构，代码的结构：

```txt
com.cr121.yuagong.biz
|--adapter
|--|--driving //primary adapter
|--|--driven  //secondary adapter
|--application
|--|--usecase //用户用例
|--|--|--command //命令类型的用户用例，会改变系统的状态
|--|--|--query //查询类的用户用例，不改变系统的状态，主要为Web页面提供各种查询
|--domain
|--|--context // 业务上下文
|--|--core
|--|--|--concepts //领域对象概念接口
|--|--|--exception //领域异常
```

一个资源的GET,POST,PUT,DELETE接口的定义在 driving 目录下的 XXXResource 类中。相关资源如果存在子资源，必须新建一个类来描述定义。对接受的请求的初步处理会分别在 command 的 XXXService 和 query 的 XXXGetService 中执行。查询建议不进入领域而直接对接数据库 mapper 操作。其余改变状态的操作在 context 中完成操作。
