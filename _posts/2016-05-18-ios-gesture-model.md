---
layout:   post
title:    iOS手势识别模型
date:     2016 May 18 20:55:33
category: iOS
tags:     iOS 手势识别 状态机 UIGestureRecognizer
---

	iOS的手势识别模型其实是一个状态机

### 手势识别状态

- 所有手势识别从一个可能状态(UIGestureRecognizerStatePossible)开始，然后开始分析、识别手势
- 如果识别失败，进入失败状态(UIGestureRecognizerStateFailed)。
- 如果识别成功，进入成功状态(UIGestureRecognizerStateRecognized)

### 连续性手势识别

- 对于连续性的手势，手势识别从Possible进入Began(UIGestureRecognizerStateBegan) ，然后会进入Change (UIGestureRecognizerStateChanged)状态，并在Change状态循环，在最后用户手指离开屏幕的时候会进入End(UIGestureRecognizerStateEnded)状态
- 连续手势也会从Change进入cancel (UIGestureRecognizerStateCancelled)，如果判断手势已经不符合该要求了。
- 每次Gesture recognizer改变状态的时候都会发送一个action message到target，知道Failed或者Cancel。

### 手势符合多个条件，

- 有时候会出现iOS判断该手势符合多个条件，而用户此时只想要其中一种，这个情况下可以调用 requireGestureRecognizerToFail: ，该方法会让当前手势对象根据指定对象状态来进行下一步操作。
    - 如果指定对象进入Begin状态，当前对象直接进入Failed；
    - 如果指定对象进入Failed或Cancel状态，当前对象会进入Begin状态。
    - 这里就涉及到了不同手势之间执行的问题。（如果想要单击和双击都要执行，并分别执行不同的逻辑，这里可以先让单机等待双击失败在执行，但这样会导致单击事件稍微延迟执行，因为单击事件需要等待双击事件失败）。

### 选择不接收、分析手势事件

- 通过 gestureRecognizer:shouldReceiveTouch: 在最开始的时候允许、拒绝接收手势事件
    - 如果想延后判断则可以通过 gestureRecognizerShouldBegin:
    - 如果返回No则会导致Failed事件

### 同时接收多个事件

- 如果想要同时接收两个事件，就调用gestureRecognizer:shouldRecognizeSimultaneouslyWithGestureRecognizer: 
    - 默认这个方法返回No，返回Yes则运行同时运行

### 事件相互作用

- 单向影响：事件A和B， 如果想要让A影响B，但不想让B影响A，
    - 通过重载方法 canPreventGestureRecognizer: 或者canBePreventedByGestureRecognizer:实现，比如
    - [rotationGestureRecognizer canPreventGestureRecognizer:pinchGestureRecognizer];

### Touch事件

- Touch事件传递过程是上到下，也就是从App->Window->View的过程
- UIWindow会让GestureRecognizer优先分析touch对象，如果GestureRecognizer成功识别这些对象，Touch事件就不会执行。
- 总结就是GestureRecognizer的优先级会高于Touch事件

### 影响Touch事件传递的方法

- delaysTouchesBegan 阻止在Begin阶段Windows交付Touch事件
- delaysTouchesEnded Window不会在Enable阶段交付Touch事件

### Gesture Recognizer是iOS定制的一些Touch事件类型，Dev可以订制自己的Gesture Recognizer，这里要继承UIGestureRecognizer并覆盖主要的四个类。
