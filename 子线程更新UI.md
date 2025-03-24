
ViewRootImpl 的创建在 onResume 方法回调之后，而我们一开篇是在 onCreate 方法中创建了子线程并访问 UI，在那个时刻，ViewRootImpl 还没有创建，我们在子线程调用 了 ImageView#setImageResource，虽然可能会触发 View#requestLayout 和 View#invalidate() ，但是由于 ViewRootImpl还未创建出来，因此 ViewRootImpl#checkThread 没有被调用到，也就是说，检测当前线程是否是创建的 UI 那个线程 的逻辑没有执行到，所以程序没有崩溃一样能跑起来。而之后修改了程序，让线程休眠了 300 毫秒后，程序就崩了。很明显 300 毫秒后 ViewRootImpl 已经创建了，可以执行 checkThread 方法检查当前线程。
开篇的例子中我们在 onCreate 方法中创建的子线程访问 UI 是一种极端的情况。实际开发中不会这么做。
