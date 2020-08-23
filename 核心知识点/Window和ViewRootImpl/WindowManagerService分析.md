
### WindowManagerService分析

### addWindow

```java
1、获取DisplayContent --- window需要在哪块显示屏上

2、判断新window的type值是否处于子窗口范围，若是则获取父窗口parentWindow

3、获取windowToken（a）。（有父窗口就拿父窗口的，否则就拿传入的属性中的）

4、若步骤3拿到的token为null，则根据rootType和type不同情况返回不同的错误原因

5、若父窗口是应用级窗口，则将步骤3的token转换为appWindowToken并赋值为atoken。根据atoken不同的情况返回不同的错误原因。

6、接下来根据token和rootType不同的异常情况返回不同的错误原因。（在type未TOAST类型时，需要给addToastWindowRequiresToken变量赋值。）

7、一系列判断过后，创建当前窗口的WindowState（b）。

8、通过DisplayContent获取显示策略。

9、判断新窗口是否要输入通道。

执行至此，从此开始，没有异常或错误发生

10、win.attach()，调用WindowState的attach，将当前窗口的包名添加到mSession（c）集合中。

11、WMS中的mWindowMap集合添加新窗口，key是该窗口客户端的Binder。

12、win.mToken.addWindow(win);创建新窗口WindowState时，获得的mToken集合添加当前新窗口。

13、创建窗口动画。

14、创建窗口大小。

15、设置InsetsState（d）属性。

16、判断焦点是否改变。如果没有改变，显示屏需要重新判断输入法的目标窗口；如果变了，则显示屏需重设输入焦点窗口。最后显示屏通过输入监听InputMonitor更新输入法窗口。

17、窗口加入成功后，打印log。如果新窗口可见，则发送更新配置信息。
```

**a： WindowToken：**

代码官方释义： Container of a set of related windows in the window manager. Often this is an AppWindowToken,which is the handle for an Activity that it uses to display windows. For nested windows, there is a WindowToken created for the parent window to manage its children.

相关窗口的集合。token是个IBinder

**b： WindowState：**

代码官方释义：A window in the window manager.

用于描述一个窗口

**c：Session：**

代码官方释义： This class represents an active client session.  There is generally one Session object per process that is interacting with the window manager.

代表一个活动的客户端，通过它与WindowManager交互。（一个进程一个Session）

**d： InsetsState：**

官方代码释义：Holder for state of system windows that cause window insets for all other windows in the system.

系统窗口状态的持有者。新窗口的插入要告诉系统窗口。



### removeWindow

从**WindowManagerImpl**的**removeView()**开始

```java
1、找到需删除View的下标。

2、获取该View的ViewRootImpl（a），以及该View。

3、告诉输入法框架IMM，该View消失。

4、执行ViewRootImpl.die()方法。并加入到mDyingViews集合中。

5、线程检测（检测是否是创建View的原始线程，若不是则抛出异常），下发View与Window解除绑定消息。

6、在下发解除绑定消息中与Accessibility相关，Input相关，DisplayManager相关解除联系，并销毁图层。

7、在下发解除绑定消息中通过IWindowSession（在服务端就是Seesion）删除目标窗口。

8、在服务端最终进入到WMS执行removeWindow将窗口删除。

9、如果View的可见性变了，则重新布局。

10、再执行一次销毁图层。

11、在相关集合如mRoots，mParams，mViews，mDyingViews中删除对应的View。

12、内存处理。
```

**a： ViewRootImpl**

**官方释义：** The top of a view hierarchy, implementing the needed protocol between View and the WindowManager.  This is for the most part an internal implementation detail of {@link WindowManagerGlobal}.

介于View和WindowManager之间的协议，桥梁。也是WindowManagerGlobal内部实现的重要部分。