# Activity的生命周期和启动模式

## Activity的生命周期分析

### 典型情况下生命周期分析

**onStart和onStop与onResume和onPause的区别**

- onStart和onStop表示应用是否可见
- onResume和onPause表示应用是否处于前台

**Activity A处于前台，然后调用Activity B，Activity A的onPause和Activity B的onResume哪个方法先执行**

新Activity启动之前，栈顶的Activity需要先onPause，然后Activity才能启动（执行onCreate、onStart、onResume）

因此，为了让新Activity尽快显示出来并切换到前台，onPause方法内不能执行耗时的操作。

#### Activity大致启动流程

1. 启动Activity的请求会有Instrumentation处理，然后通过Binder向AMS（ActivityManagerService）发请求
2. AMS内部维护一个ActivityStack，并负责栈内的Activity的状态同步，AMS通过ActivityThread去同步Activity的状态从而完成生命周期方法的调用。





### 异常情况下生命周期分析

**资源相关的系统配置发生改变导致Activity被杀死并重建**

屏幕旋转时，系统配置发生改变，默认情况下Activity就会被销毁并重建

**销毁时**

==异常情况下（需要重新显示的情况下）==，会在onStop方法调用之前，执行onSaveInstanceState方法来保存当前Activity的状态，保存在Bundle对象中

**重建时**

会在onStart方法调用之后，保存在Bundle对象会作为参数被传入到onRestoreInstanceState和onCreate方法，onRestoreInstanceState方法调用时机在onStart之后。

默认情况下，onSaveInstanceState和onRestoreInstanceState中，系统已经做了一定的恢复工作（当前Activity的视图结构：文本框中用户输入的数据、ListView的位置）。

**注意**

- onCreate方法中，Bundle savedInstanceState参数不一定有值
- onRestoreInstanceState一旦被调用，Bundle savedInstanceState参数一定是有值



**防止Activity重新创建**

添加configChanges属性并设置值，该属性可以防止Activity重建，取而代之，是系统调用了Activity的onConfigurationChanged的方法

通过设置

  `android:configChanges="orientation|screenSize` 来防止屏幕旋转时重建Activity

#### 保存和恢复View层次结构

工作流程（委托模式

- Activity被意外终止时，Activity会调用onSaveInstanceState去保存数据。
- 然后Activity会委托Window去保存数据
- Window再委托它上面的容器去保存数据。顶层容器是一个ViewGroup，一般来说它很可能是DecorView
- 最后顶层容器再一一通知它的子元素去保存数据





**资源内存不足导致低优先级的Activity被杀死**

Activity按优先级分类

- 前台Activity
- 可见但非前台Activity（调用onStart之后，调用onResume之前）
- 后台Activity

如果一个进程没有四大组件在执行，该进程将很快被杀死。因此，比较好的方法是将一些后台工作放在Service中，从而保证进程有一定的优先级







## Activity启动模式分析

### standard标准模式

系统模式模式

**特点**

- 每次启动一个Activity，就会创建一个新的实例，不管Activity是否存在
- 被启动的Activity会运行在启动它的Activity所在的任务栈内
  - 直接使用ApplicationContext去启动Activity报错原因：ApplicationContext是非Activity类型的Context，并没有任务栈，所以会启动失败
  - 解决方法：隐式启动时，为待启动的Activity指定标记位 `FLAG_ACTIVITY_NEW_TASK` ，这样启动时就会创建一个新的任务栈，不过此时是以singleTask模式启动



### singleTop栈顶复用模式

**特点**

- 如果新Activity已经位于任务栈的栈顶，则Activity不会被重新创建；如果不位于栈顶，则新建实例
- 栈顶复用时，会触发onPause、onResume、onNewIntent方法，其中onNewIntent方法参数

### singleTask栈内复用模式

### singleInstance单实例模式



## IntentFilter的匹配规则



# IPC机制

