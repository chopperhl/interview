
* 普通service的默认在主线程执行的  必须手动调用stopservice 才能够关闭service

* IntentService就是一种能自己开启线程并在任务执行后自己关闭的service

IntentService 启动后会 会先执行OnStartCommand 该方法在主线程执行

接下来会执行handleIntent 该方法会子线程执行
