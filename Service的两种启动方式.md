# Service的两种启动方式

 ### Start service

 每次start 启动一个新的实列

 ### bind service

 可以设置`bind_auto_create`参数

 第一次调用`bindservice`会创建一个实例 并在serviceConnect中回调得到Binder对象

 这个binder需要Service的`onBind`方法中实现

 之后在此调用都会重新连接上这个service不会再重复创建