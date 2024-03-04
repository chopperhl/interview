# Binder机制

## 什么是Binder

Binder是安卓提供的一种跨进程通信机制，底层用binder驱动支持，通过mmap系统调用进行用户空间和内核空间的映射，用户通过向用户空间写入数据即可同步更新到内核空间，整个过程中只有一次cpoy_from_user的操作，比管道，socket等跨进程通信方案更加高效。

## 手动实现BInder java代码

```java
//Service端 serive onBind中返回这个Binder
public abstract class Stub extends android.os.Binder implements IInterface {
    private static final String DESCRIPTOR = "com.chopperhl.bindersample.MyBinder";
    public Stub() {
        this.attachInterface(this, DESCRIPTOR);
    }
    @Override
    public android.os.IBinder asBinder() {
        return this;
    }

    @Override
    public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
        String descriptor = DESCRIPTOR;
        switch (code) {
            case INTERFACE_TRANSACTION: {
                reply.writeString(descriptor);
                return true;
            }
            case TRANSACTION_queryData: {
                data.enforceInterface(descriptor);
                String _result = this.queryData();
                reply.writeNoException();
                reply.writeString(_result);
                return true;
            }
            default: {
                return super.onTransact(code, data, reply, flags);
            }
        }
    }
    static final int TRANSACTION_queryData = (android.os.IBinder.FIRST_CALL_TRANSACTION);
    public abstract String queryData() throws android.os.RemoteException;
}

//Client端 在ServiceConnection中调用asInterface转换得到Proxy
public class Proxy implements IInterface {
    private static final String DESCRIPTOR = "com.chopperhl.bindersample.MyBinder";
    private android.os.IBinder mRemote;

    Proxy(android.os.IBinder remote) {
        mRemote = remote;
    }
    public static Proxy asInterface(android.os.IBinder obj) {
        if ((obj == null)) {
            return null;
        }
        return new Proxy(obj);
    }
    @Override
    public android.os.IBinder asBinder() {
        return mRemote;
    }
    public String getInterfaceDescriptor() {
        return DESCRIPTOR;
    }
    public String queryData() throws android.os.RemoteException {
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        String _result;
        try {
            _data.writeInterfaceToken(DESCRIPTOR);
            boolean _status = mRemote.transact(TRANSACTION_queryData, _data, _reply, 0);
            _reply.readException();
            _result = _reply.readString();
        } finally {
            _reply.recycle();
            _data.recycle();
        }
        return _result;
    }
    static final int TRANSACTION_queryData = (android.os.IBinder.FIRST_CALL_TRANSACTION);
}
```

*官方aidl只是一种数据规范，用来约定Parcel，和方法名指向，方便别人对接，或者制作sdk提供给调用方*

## 其他

四大组件的进程通讯底层都是基于Binder来实现的，Intent不能传递大数据的本质是调用mmap的时候设置了最大空间（1m-8k），事件使用中，由于我们的原始数据要经过一层包装和序列化，实际传输的数据应该小于这个值。

*todo扩展*
