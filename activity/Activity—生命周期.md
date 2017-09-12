# Activity—生命周期

**作为android四大组件，activity的作用对于每一个android开发来说都是非凡的。之前学习android，只知道7个生命周期，但是具体的详细的东西却不求甚解，这篇主要是深入了解一下生命周期，重新学习一下这个熟悉的activity。**

![](<image/activity_lifecycle.png>)

上图是最基础的activity的生命周期的一个调用的过程

### 一、生命周期中各个方法的含义和作用

#### 1、7个生命周期的含义

#### （1）onCreate:

​	它本身的作用是进行Activity的一些初始化工作，比如使用setContentView加载布局，对一些控件和变量进行初始化等。此时**Activity还在后台，不可见**。所以动画不应该在这里初始化，因为看不到……

#### （2）onStart：

​	此时Activity已经**可见**了，但是还没出现在前台，我们还看不到，无法与Activity交互。其实将Activity的初始化工作放在这也没有什么问题，放在onCreate中是由于官方推荐的以及我们开发的习惯。

#### （3）onResume:

​	此时Activity经过前两个阶段的初始化已经蓄势待发。Activity在这个阶段已经出现在前台并且**可见**了。这个阶段可以打开独占设备

#### （4）onPause:

​	当Activity要跳到另一个Activity或应用正常退出时都会执行这个方法。此时Activity在前台并**可见**，我们可以进行一些轻量级的存储数据和去初始化的工作，不能太耗时，因为在跳转Activity时只有当一个Activity执行完了onPause方法后另一个Activity才会启动，而且android中指定如果onPause在500ms即0.5秒内没有执行完毕的话就会强制关闭Activity。

#### （5）onStop：

​	此时Activity已经**不可见**了，但是Activity对象还在内存中，没有被销毁。这个阶段的主要工作也是做一些资源的回收工作。

#### （6）onDestroy：

​	这个阶段Activity被销毁，**不可见**，我们可以将还没释放的资源释放，以及进行一些回收工作。

#### （7）onRestart：

​	Activity在这时**可见**，当用户按Home键切换到桌面后又切回来或者从后一个Activity切回前一个Activity就会触发这个方法。这里一般不做什么操作。

#### 2、各个生命周期的区别

### onCreate和onStart之间有什么区别？

（1）可见与不可见的区别。前者不可见，后者可见。
（2）执行次数的区别。onCreate方法只在Activity创建时执行**一次**，而onStart方法在Activity的切换以及按Home键返回桌面再切回应用的过程中被多次调用。因此**Bundle数据的恢复在onStart中进行比onCreate中执行更合适**。
（3）onCreate能做的事onStart其实都能做，但是onstart能做的事onCreate却未必适合做。如前文所说的，setContentView和资源初始化在两者都能做，然而想动画的初始化在onStart中做比较好。

### onStart方法和onResume方法有什么区别？

（1）是否在前台。onStart方法中Activity可见但不在前台，不可交互，而在onResume中在前台。
（2）职责不同，onStart方法中主要还是进行初始化工作，而onResume方法，根据官方的建议，可以做开启动画和独占设备的操作。

### onPause方法和onStop方法有什么区别？

（1）是否可见。onPause时Activity可见，onStop时Activity不可见，**但Activity对象还在内存中**。
（2）在系统内存不足的时候可能不会执行onStop方法，因此程序状态的保存、独占设备和动画的关闭、以及一些数据的保存最好在onPause中进行，但要注意不能太耗时。

### onStop方法和onDestroy方法有什么区别？

onStop阶段Activity还没有被销毁，对象还在内存中，此时可以通过切换Activity再次回到该Activity，而onDestroy阶段Acivity被销毁

### 6.为什么切换Activity时各方法的执行次序是(A)onPause→(B)onCreate→(B)onStart→(B)onResume→(A)onStop而不是(A)onPause→(A)onStop→(B)onCreate→(B)onStart→(B)onResume

（1）一个Activity或多或少会占有系统资源，而在官方的建议中，onPause方法将会释放掉很多系统资源，为切换Activity提供流畅性的保障，而不需要再等多两个阶段，这样做切换更快。
（2）按照生命周期图的表示，如果用户在切换Activity的过程中再次切回原Activity，是在onPause方法后直接调用onResume方法的，这样比onPause→onStop→onRestart→onStart→onResume要快得多。

### onSaveInstanceState和onRestoreInstanceState

#### **onSaveInstanceState**

(1)在Activity被覆盖或退居后台之后，系统资源不足将其杀死，此方法会被调用；

(2)在用户改变屏幕方向时，此方法会被调用；

(3)在当前Activity跳转到其他Activity或者按Home键回到主屏，自身退居后台时，此方法会被调用。

第一种情况我们无法保证什么时候发生，系统根据资源紧张程度去调度；

第二种是屏幕翻转方向时，系统先销毁当前的Activity，然后再重建一个新的，调用此方法时，我们可以保存一些临时数据；

第三种情况系统调用此方法是为了保存当前窗口各个View组件的状态。**onSaveInstanceState的调用顺序是在onPause之前。**

#### **onRestoreInstanceState**

(1)在Activity被覆盖或退居后台之后，系统资源不足将其杀死，然后用户又回到了此Activity，此方法会被调用；

(2)在用户改变屏幕方向时，重建的过程中，此方法会被调用。我们可以重写此方法，以便可以恢复一些临时数据。onRestoreInstanceState的调用顺序是在onStart之后。

### 三、具体过程

`package com.scott.lifecycle;`  

  

import android.app.Activity;  

import android.content.res.Configuration;  

import android.os.Bundle;  

import android.util.Log;  

  

public class OrientationActivity extends Activity {  

​      

​    private static final String TAG = "OrientationActivity";  

​    private int param = 1;  

​      

​    @Override  

​    protected void onCreate(Bundle savedInstanceState) {  

​        super.onCreate(savedInstanceState);  

​        setContentView(R.layout.orientation_portrait);  

​        Log.i(TAG, "onCreate called.");  

​    }  

​      

​    @Override  

​    protected void onStart() {  

​        super.onStart();  

​        Log.i(TAG, "onStart called.");  

​    }  

​      

​    @Override  

​    protected void onRestart() {  

​        super.onRestart();  

​        Log.i(TAG, "onRestart called.");  

​    }  

​      

​    @Override  

​    protected void onResume() {  

​        super.onResume();  

​        Log.i(TAG, "onResume called.");  

​    }  

​      

​    @Override  

​    protected void onPause() {  

​        super.onPause();  

​        Log.i(TAG, "onPause called.");  

​    }  

​      

​    @Override  

​    protected void onStop() {  

​        super.onStop();  

​        Log.i(TAG, "onStop called.");  

​    }  

​      

​    @Override  

​    protected void onDestroy() {  

​        super.onDestroy();  

​        Log.i(TAG, "onDestory called.");  

​    }  

  

​    @Override  

​    protected void onSaveInstanceState(Bundle outState) {  

​        outState.putInt("param", param);  

​        Log.i(TAG, "onSaveInstanceState called. put param: " + param);  

​        super.onSaveInstanceState(outState);  

​    }  

​      

​    @Override  

​    protected void onRestoreInstanceState(Bundle savedInstanceState) {  

​        param = savedInstanceState.getInt("param");  

​        Log.i(TAG, "onRestoreInstanceState called. get param: " + param);  

​        super.onRestoreInstanceState(savedInstanceState);  

​    }  

​      

​    //当指定了android:configChanges="orientation"后,方向改变时onConfigurationChanged被调用  

​    @Override  

​    public void onConfigurationChanged(Configuration newConfig) {  

​        super.onConfigurationChanged(newConfig);  

​        Log.i(TAG, "onConfigurationChanged called.");  

​        switch (newConfig.orientation) {  

​        case Configuration.ORIENTATION_PORTRAIT:  

​            setContentView(R.layout.orientation_portrait);  

​            break;  

​        case Configuration.ORIENTATION_LANDSCAPE:  

​            setContentView(R.layout.orientation_landscape);  

​            break;  

​        } 

​    }  

`}`  

1.启动Activity：

![](<image/activitylife1.png>)

在系统调用了onCreate和onStart之后，调用了onResume，自此，Activity进入了运行状态。

2.跳转到其他Activity，或按下Home键回到主屏：

![](<image/activitylife2.gif>)

我们看到，此时onSaveInstanceState方法在onPause之前被调用了，并且注意，退居后台时，onPause后onStop相继被调用。

3.从后台回到前台：

![](<image/activitylife3.gif>)

当从后台会到前台时，系统先调用onRestart方法，然后调用onStart方法，最后调用onResume方法，Activity又进入了运行状态。

4.修改TargetActivity在AndroidManifest.xml中的配置，将android:theme属性设置为@android:style/Theme.Dialog，然后再点击LifeCycleActivity中的按钮，跳转行为就变为了TargetActivity覆盖到LifeCycleActivity之上了，此时调用的方法为：

![](<image/activitylife4.gif>)

注意还有一种情况就是，我们点击按钮，只是按下锁屏键，执行的效果也是如上。

我们注意到，此时LifeCycleActivity的OnPause方法被调用，并没有调用onStop方法，因为此时的LifeCycleActivity没有退居后台，只是被覆盖或被锁屏；onSaveInstanceState会在onPause之前被调用。

5.按回退键使LifeCycleActivity从被覆盖回到前面，或者按解锁键解锁屏幕：

![](<image/activitylife5.gif>)

此时只有onResume方法被调用，直接再次进入运行状态。

6.退出：

![](<image/activitylife6.gif>)

最后onDestory方法被调用，标志着LifeCycleActivity的终结。

大家似乎注意到，在所有的过程中，并没有onRestoreInstanceState的出现，这个并不奇怪，因为之前我们就说过，onRestoreInstanceState只有在杀死不在前台的Activity之后用户回到此Activity，或者用户改变屏幕方向的这两个重建过程中被调用。我们要演示第一种情况比较困难，我们可以结合第二种情况演示一下具体过程。顺便也向大家讲解一下屏幕方向改变的应对策略。

7.旋转屏幕

![](<image/activitylife7.gif>)

系统先是调用onSaveInstanceState方法，我们保存了一个临时参数到Bundle对象里面，然后当Activity重建之后我们又成功的取出了这个参数。

为了避免这样销毁重建的过程，我们需要在AndroidMainfest.xml中对OrientationActivity对应的<activity>配置android:configChanges="orientation"，然后我们再测试一下，我试着做了四次的旋转，打印如下：

![](<image/activitylife7.gif>)

可以看到，每次旋转方向时，只有onConfigurationChanged方法被调用，没有了销毁重建的过程。

以下是需要注意的几点：

1.如果<activity>配置了android:screenOrientation属性，则会使android:configChanges="orientation"失效。

2.模拟器与真机差别很大：模拟器中如果不配置android:configChanges属性或配置值为orientation，切到横屏执行一次销毁->重建，切到竖屏执行两次。真机均为一次。模拟器中如果配置android:configChanges="orientation|keyboardHidden"（如果是Android4.0，则是"orientation|keyboardHidden|screenSize"），切竖屏执行一次onConfigurationChanged，切横屏执行两次。真机均为一次。

Activity的生命周期与程序的健壮性有着密不可分的关系，希望朋友们能够认真体会、熟练应用。