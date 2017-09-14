# Activity的四大启动模式

> 在android开发，activity的跳转是最常用的功能，启动模式可以说是敲门的方式，这一节我们好好总结一下activity的四大敲门方式。

## 一、基础概念

*在开始介绍启动模式之前，我们先了解一下相关的一些基本概念，不然后面会一脸懵逼*

### 1.任务栈

任务栈Task，是一种用来放置Activity实例的容器，他是以**栈**的形式进行盛放，也就是所谓的**先进后出**，主要有2个基本操作：**压栈和出栈**，其所存放的Activity是不支持重新排序的，只能根据压栈和出栈操作更改Activity的顺序。

启动一个Application的时候，系统会为它默认创建一个对应的Task，用来放置根Activity。默认启动Activity会放在同一个Task中，新启动的Activity会被压入启动它的那个Activity的栈中，并且显示它。当用户按下回退键时，这个Activity就会被弹出栈，按下Home键回到桌面，再启动另一个应用，这时候之前那个Task就被移到后台，成为后台任务栈，而刚启动的那个Task就被调到前台，成为前台任务栈，Android系统显示的就是前台任务栈中的Top实例Activity。

### 2.任务栈的作用

以往基于应用（application）的程序开发中，程序具有明确的边界，一个程序就是一个应用，一个应用为了实现功能可以采用开辟新线程甚至新进程来辅助，但是应用与应用之间不能复用资源和功能。而Android引入了基于**组件开发**的软件架构，虽然我们开发android程序，仍然使用一个apk工程一个Application的开发形式，但是对于Aplication的开发就用到了Activity、service等四大组件，其中的每一个组件，都是可以被**跨应用复用**的，这就是android的神奇之处。虽然组件可以跨应用被调用，但是一个组件所在的进程必须是在组件所在的Aplication进程中。由于android强化了组件概念，弱化了Aplication的概念，所以在android程序开发中，A应用的A组件想要使用拍照或录像的功能就可以不用去针对Camera类进行开发，直接调用系统自带的摄像头应用（称其B应用）中的组件（称其B组件）就可以了，但是这就引发了一个新问题，A组件运行在A应用中，B组件运行在B应用中，自然都不在同一个进程中，那么从B组件中返回的时候，如何实现正确返回到A组件呢？Task就是来负责实现这个功能的，它是从用户角度来理解应用而建立的一个抽象概念。因为用户所能看到的组件就是Activity，所以Task可以理解为实现一个功能而负责管理所有用到的Activity实例的栈。

栈是一个**先进后出**的线性表，根据Activity在当前栈结构中的位置，来决定该Activity的状态。正常情况下，当一个Activity启动了另一个Activity的时候，新启动的Activity就会置于任务栈的顶端，并处于活动状态，而启动它的Activity虽然成功身退，但依然保留在任务栈中，处于停止状态，当用户按下返回键或者调用finish()方法时，系统会移除顶部Activity，让后面的Activity恢复活动状态。当然，世界不可能一直这么“和谐”，可以给Activity设置一些“特权”，来打破这种“和谐”的模式，这种特权，就是通过在AndroidManifest文件中的属性andorid:launchMode来设置或者通过Intent的flag来设置的，下面就先介绍下Activity的几种启动模式。

### 3.taskAffinity

- 这个参数标识了一个Activity所需任务栈的名字，默认情况下，所有Activity所需的任务栈的名字为应用的包名
- 我们可以单独指定每一个Activity的taskAffinity属性覆盖默认值
- 一个任务的affinity决定于这个任务的根activity（root activity）的taskAffinity
- 在概念上，具有相同的affinity的activity（即设置了相同taskAffinity属性的activity）属于同一个任务
- 为一个activity的taskAffinity设置一个空字符串，表明这个activity不属于任何task

### 二、四大启动模式

#### 1.standard

**默认模式**

可以不用写配置。在这个模式下，都会默认创建一个新的实例。因此，在这种模式下，可以有多个相同的实例，也允许多个相同Activity叠加。

应用场景：**绝大多数Activity**。

注意：*如果以这种方式启动的Activity被**跨进程调用**，在5.0之前新启动的Activity实例会放入发送Intent的Task的栈的顶部，尽管它们属于不同的程序，这似乎有点费解看起来也不是那么合理，所以在5.0之后，上述情景会创建一个新的Task，新启动的Activity就会放入刚创建的Task中，这样就合理的多了。*

#### 2.**singleTop**

**栈顶复用模式**

如果要开启的activity在任务栈的顶部已经存在，就不会创建新的实例，而是调用 onNewIntent() 方法。避免栈顶的activity被重复的创建。

应用场景：在通知栏点击收到的通知，然后需要启动一个Activity，这个Activity就可以用singleTop，否则每次点击都会新建一个Activity。当然实际的开发过程中，测试妹纸没准给你提过这样的bug：某个场景下连续快速点击，启动了两个Activity。如果这个时候待启动的Activity使用 singleTop模式也是可以避免这个Bug的。

**singleTop模式分3种情况**

1. 当前栈中已有该Activity的实例并且该实例位于栈顶时，不会新建实例，而是复用栈顶的实例，并且会将Intent对象传入，回调onNewIntent方法
2. 当前栈中已有该Activity的实例但是该实例不在栈顶时，其行为和standard启动模式一样，依然会创建一个新的实例
3. 当前栈中不存在该Activity的实例时，其行为同standard启动模式

注意：*standard和singleTop启动模式都是在原任务栈中新建Activity实例，不会启动新的Task，即使你指定了taskAffinity属性。* 

#### 3.**singleTask**

#### **栈内复用模式**

activity只会在任务栈里面存在一个实例。如果要激活的activity，在任务栈里面已经存在，就不会创建新的activity，而是复用这个已经存在的activity，调用 onNewIntent() 方法，并且清空这个activity任务栈上面所有的activity。其实这个过程还存在一个任务栈的匹配，因为这个模式启动时，会在自己需要的任务栈中寻找实例，这个任务栈就是通过taskAffinity属性指定。如果这个任务栈不存在，则会创建这个任务栈。 

应用场景：大多数App的主页。对于大部分应用，当我们在主界面点击回退按钮的时候都是退出应用，那么当我们第一次进入主界面之后，主界面位于栈底，以后不管我们打开了多少个Activity，只要我们再次回到主界面，都应该使用将主界面Activity上所有的Activity移除的方式来让主界面Activity处于栈顶，而不是往栈顶新加一个主界面Activity的实例，通过这种方式能够保证退出应用时所有的Activity都能报销毁。
在跨应用Intent传递时，如果系统中不存在singleTask Activity的实例，那么将创建一个新的Task，然后创建SingleTask Activity的实例，将其放入新的Task中。

#### 4.**singleInstance**

**单一实例模式**

整个手机操作系统里面只有一个实例存在。不同的应用去打开这个activity  共享公用的同一个activity。他会运行在自己单独，独立的任务栈里面，并且任务栈里面只有他一个实例存在。

应用场景：呼叫来电界面。这种模式的使用情况比较罕见，在Launcher中可能使用。或者你确定你需要使Activity只有一个实例。建议谨慎使用。

### 三、设置方式

系统提供了两种方式来设置一个Activity的启动模式，除了在AndroidManifest文件中设置以外，还可以通过Intent的Flag来设置一个Activity的启动模式，下面我们在简单介绍下一些Flag。

**FLAG_ACTIVITY_NEW_TASK**

使用一个新的Task来启动一个Activity，但启动的每个Activity都讲在一个新的Task中。与指定android：launchMode=“singleInstance”效果相同

**该Flag通常使用在从Service中启动Activity的场景**，由于Service中并不存在Activity栈，所以使用该Flag来创建一个新的Activity栈，并创建新的Activity实例。

**FLAG_ACTIVITY_SINGLE_TOP**

使用singletop模式启动一个Activity，与指定android：launchMode=“singleTop”效果相同。

**FLAG_ACTIVITY_CLEAR_TOP**

使用SingleTask模式来启动一个Activity，与指定android：launchMode=“singleTask”效果相同。

**FLAG_ACTIVITY_NO_HISTORY**

Activity使用这种模式启动Activity，当该Activity启动其他Activity后，该Activity就消失了，不会保留在Activity栈中。

注意：**LaunchMode与StartActivityForResult**

我们在开发过程中经常会用到StartActivityForResult方法启动一个Activity，然后在onActivityResult()方法中可以接收到上个页面的回传值，但你有可能遇到过拿不到返回值的情况，那有可能是因为Activity的LaunchMode设置为了singleTask。5.0之后，android的LaunchMode与StartActivityForResult的关系发生了一些改变。

这是因为ActivityStackSupervisor类中的startActivityUncheckedLocked方法在5.0中进行了修改。在5.0之前，当启动一个Activity时，系统将首先检查Activity的launchMode，如果为A页面设置为SingleInstance或者B页面设置为singleTask或者singleInstance,则会在LaunchFlags中加入FLAG_ACTIVITY_NEW_TASK标志，而如果含有FLAG_ACTIVITY_NEW_TASK标志的话，onActivityResult将会立即接收到一个cancle的信息，而5.0之后这个方法做了修改，修改之后即便启动的页面设置launchMode为singleTask或singleInstance，onActivityResult依旧可以正常工作，也就是说无论设置哪种启动方式，StartActivityForResult和onActivityResult()这一组合都是有效的。所以如果你目前正好基于5.0做相关开发，不要忘了向下兼容，这里有个坑请注意避让。

本文参考阅读：

http://www.jianshu.com/p/2a9fcf3c11e4

http://blog.csdn.net/mynameishuangshuai/article/details/51491074