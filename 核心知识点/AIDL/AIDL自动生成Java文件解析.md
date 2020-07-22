[TOC]

## 成员结构

- **Parcel**

```java
//官方解释
//Container for a message (data and object references) that can be sent through an IBinder. 
```

通过IBinder传输的消息的一种封装。（数据封装）

- **IBinder**

```java
//官方解释
//Base interface for a remotable object, the core part of a lightweight remote procedure call mechanism designed for high performance when performing in-process and cross-process calls.
```

基于远程对象的接口，轻量级远程处理调用机制的核心部分，这个调用机制是为了在进程间和进程内高效运行而设计的。（即IBinder使一套接口规范，描述了调用远程对象的基本方式）

**Binder**就是该接口的实现类。

- **IInterface**

Binder接口的基类，若定义新的接口时，需要继承该接口。

## Java文件解读

### 结构

```java
public interface AAA extends android.os.IInterface //1
{
/** Local-side IPC implementation stub class. */
public static abstract class Stub extends android.os.Binder implements com.wx.demo.AAA //2
{
 ......
}
/**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
public void aaa(java.lang.String data) throws android.os.RemoteException; //3
}
```

如上，是当我们新建AIDL文件后，编译项目时，AS自动为我们创建的Java文件基本结构：

- 注释1，新建的Binder接口，需要继承**IInterface**；
- 注释2，是一个继承自**Binder**，并实现**AAA**新建的**Binder**接口类的内部类。（对Binder的相关操作均在此处）；
- 注释3，暴露给客户端的接口方法。由于**Stub**实现**AAA**接口，所以该方法的实现在**Stub**内；

以上是一个AIDL对应的Java文件的基本结构。下面主要分析分析Stub的内部构造

### Stub

```java
/** Local-side IPC implementation stub class. */
public static abstract class Stub extends android.os.Binder implements com.wx.demo.IMyServerAidl
{
private static final java.lang.String DESCRIPTOR = "com.wx.demo.IMyServerAidl"; //1
/** Construct the stub at attach it to the interface. */
public Stub()
{
this.attachInterface(this, DESCRIPTOR);
}
/**
 * Cast an IBinder object into an com.wx.demo.IMyServerAidl interface,
 * generating a proxy if needed.
 */
public static com.wx.demo.IMyServerAidl asInterface(android.os.IBinder obj) //2
{
if ((obj==null)) {
return null;
}
android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR); //3
if (((iin!=null)&&(iin instanceof com.wx.demo.IMyServerAidl))) {
return ((com.wx.demo.IMyServerAidl)iin);
}
return new com.wx.demo.IMyServerAidl.Stub.Proxy(obj); //4
}
@Override public android.os.IBinder asBinder()
{
return this;
}
@Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException //5
{
java.lang.String descriptor = DESCRIPTOR;
switch (code)
{
case INTERFACE_TRANSACTION:
{
reply.writeString(descriptor);
return true;
}
case TRANSACTION_reportData:
{
data.enforceInterface(descriptor); //6
java.lang.String _arg0;
_arg0 = data.readString();
this.reportData(_arg0); //7
reply.writeNoException();
return true;
}
default:
{
return super.onTransact(code, data, reply, flags);
}
}
}
private static class Proxy implements com.wx.demo.IMyServerAidl //8
{
 ......
}
static final int TRANSACTION_reportData = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0); //9
}
​```

```

首先，注释1是一个final型的字符串，称作描述符。它的作用个人理解是给当前在IPC中传输的数据封装Parcel做一个标记，以便在提取时知道要获取哪一个Parcel的数据。 

注释2，asInterface()方法，用过AIDL的话应该不会陌生，这是我们获取服务端Binder引用的方法。传入的既是服务绑定时的IBinder。若传入null的话，则直接返回null。若不为null，则首先通过注释3，通过该IBinder的**queryLocalInterface**方法查询本地是否有该接口实现，若有则直接返回；若没有，则进入注释4，返回Stub内部类的Proxy对象。（**Proxy**内容下文再介绍）

然后是另一个重要方法**onTransact**。注释5，是对**onTransact**方法的实现。在onTransact中主要完成如下事件：

- 获取全局的DESCRIPTION描述符；
- 根据传入的code，判断客户端是调用的哪个方法
- 若是调用暴露给客户端的接口方法，则首先通过注释6的方法，通过描述符拿到Parcel数据，然后获取客户端传来的参数，并将其作为参数传入注释7方法，此时，调用服务端的实现方法。即在服务中继承Stub的自定义类中。

最后注释9，是对该方法的code进行的赋值操作。

#### 小结

**Stub**主要完成如下几件事：

- 定义DESCRIPTION描述符；
- 提供客户端获取服务端Binder引用的方法；
- 实现服务端处理客户端调用的onTransact方法的实现
- 定义相关方法调用的code值；

### Proxy

最后来介绍一下**Binder**代理---**Proxy**。

```java
private static class Proxy implements com.wx.demo.IMyServerAidl
{
private android.os.IBinder mRemote; //1
Proxy(android.os.IBinder remote) //2
{
mRemote = remote;
}
@Override public android.os.IBinder asBinder()
{
return mRemote;
}
public java.lang.String getInterfaceDescriptor()
{
return DESCRIPTOR;
}
@Override public void reportData(java.lang.String data) throws android.os.RemoteException//3
{
android.os.Parcel _data = android.os.Parcel.obtain();//4
android.os.Parcel _reply = android.os.Parcel.obtain();//5
try {
_data.writeInterfaceToken(DESCRIPTOR);//6
_data.writeString(data);
mRemote.transact(Stub.TRANSACTION_reportData, _data, _reply, 0);//7
_reply.readException();
}
finally {//8
_reply.recycle();
_data.recycle();
}
}
}
```

首先在注释1，定义了一个IBinder变量**mRemote**。其会在注释2时的构造方法中进行赋值。在介绍Stub时，获取服务端Binder引用中，当未在本地查询到实现，则返回Proxy对象时就调用了该构造方法，并传入了远程服务（即客户端的引用）。

然后，注释3，是服务端相应接口方法的实现。首先注释4，获取Parcel对象实例，作为要传的数据；然后是注释5，获取另一个Parcel对象实例，作为回复的数据。

接着，注释6，记录当前Parcel的描述符，然后，写入要传递的数据，最后在注释7，通过IBinder的**transact**方法将数据传送出去。这里在传入时，为第一个参数code设定了值，那么服务端onTransact方法中就会根据该code值进行处理。

最后，注释8对资源进行释放回收操作。

#### 小结

- 记录客户端的引用；
- 实现相应接口方法，并利用客户端引用通过transact方法传输出去；

### 总结

AIDL的传输过程，就是客户端的"组包"，然后通过transact()方法传输给服务端；服务端对其"解包"后，通过onTransact()方法进行处理。