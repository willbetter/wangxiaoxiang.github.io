---
layout:     keynote
title:      "UIViewController转场动画"
date:       2015-09-16
author:     "GoTT"
header-img: "img/post-bg-2015.jpg"
category : [UIViewController,UICollectionView,转场动画,iOS动画]
tags: iOS

---

[建设中...]

资源整理：

* [LookInside](developer.apple.com/library/etc/redirect/xcode/ios/1048/samplecode/LookInside/Introduction/Intro.html)
* [CollectionViewTransition](https://developer.apple.com/library/prerelease/ios/samplecode/CollectionViewTransition/Introduction/Intro.html)

# 前言
在iOS7之前自定义UIViewController之间的切换可能需要如下这样，但是这样使用会很麻烦。

```
[self addChildViewController:toVC];
[fromVC willMoveToParentViewController:nil];
[self.view addSubview:toVC.view];

__weak id weakSelf = self;  
[self transitionFromViewController:fromVC
                  toViewController:toVC duration:0.3
                           options:UIViewAnimationOptionTransitionCrossDissolve
                        animations:^{}
                        completion:^(BOOL finished) {
    [fromVC.view removeFromSuperView];
    [fromVC removeFromParentViewController];
    [toVC didMoveToParentViewController:weakSelf];
}];
```

幸好iOS7开始苹果给我们提供了新的API。在深入了解之前先来了解API

# iOS7新增API

##### 基础动画

这个接口用来提供切换上下文给开发者使用，包含了从哪个ViewController到哪个ViewController等各类信息，一般不需要开发者自己实现。具体来说，iOS7的自定义切换目的之一就是切换相关代码解耦，在进行VC切换时，做切换效果实现的时候必须要需要切换前后VC的一些信息，系统在新加入的API的比较的地方都会提供一个实现了该接口的对象，以供我们使用。

```
@protocol UIViewControllerContextTransitioning <NSObject>

/*
 *  ViewController切换所发生的view容器，开发者应该将切出的view移除，将切入的view加入到该view容器中。
 */
- (nullable UIView *)containerView;

- (BOOL)isAnimated;

- (BOOL)isInteractive;

- (BOOL)transitionWasCancelled;

- (UIModalPresentationStyle)presentationStyle;

- (void)updateInteractiveTransition:(CGFloat)percentComplete;

- (void)finishInteractiveTransition;

- (void)cancelInteractiveTransition;

/*
 *  向这个context报告切换已经完成。
 */
- (void)completeTransition:(BOOL)didComplete;

/*
 *  提供一个key，返回对应的UIViewController。现在的SDK中key的选择只有UITransitionContextFromViewControllerKey和UITransitionContextToViewControllerKey两种，分别表示将要切出和切入的UIViewController。
 */
- (nullable __kindof UIViewController *)viewControllerForKey:(NSString *)key;

/*
 *  提供一个key，返回对应的UIViewController的View。现在的SDK中key的选择只有UITransitionContextFromViewKey和UITransitionContextToViewKey两种，分别表示将要切出和切入的UIViewController的view。
 */
- (nullable __kindof UIView *)viewForKey:(NSString *)key NS_AVAILABLE_IOS(8_0);

- (CGAffineTransform)targetTransform NS_AVAILABLE_IOS(8_0);

/*
 *  某个VC的初始位置，可以用来做动画的计算。
 */
- (CGRect)initialFrameForViewController:(UIViewController *)vc;

/*
 *  某个VC的最终位置，可以用来做动画的计算。
 */
- (CGRect)finalFrameForViewController:(UIViewController *)vc;

@end
```

###### UIViewControllerAnimatedTransitioning

这个接口负责切换的具体内容，也即“切换中应该发生什么”。开发者在做自定义切换效果时大部分代码会是用来实现这个接口。它只有两个方法需要我们实现：

```
@protocol UIViewControllerAnimatedTransitioning <NSObject>

/*
 *  系统给出一个切换上下文，我们根据上下文环境返回这个切换所需要的花费时间（一般就返回动画的时间就好了，SDK会用这个时间来在百分比驱动的切换中进行帧的计算，后面再详细展开）。
 */
- (NSTimeInterval)transitionDuration:(nullable id <UIViewControllerContextTransitioning>)transitionContext;

/*
 *  在进行切换的时候将调用该方法，我们对于切换时的UIView的设置和动画都在这个方法中完成。
 */
- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext;


@optional

// This is a convenience and if implemented will be invoked by the system when the transition context's completeTransition: method is invoked.
- (void)animationEnded:(BOOL) transitionCompleted;

@end
```

###### UIViewControllerTransitioningDelegate

这个接口的作用比较简单单一，在需要ViewController切换的时候系统会像实现了这个接口的对象询问是否需要使用自定义的切换效果。这个五接口共有四个类似的方法：

```
@protocol UIViewControllerTransitioningDelegate <NSObject>

@optional
- (nullable id <UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *)source;

- (nullable id <UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed;

- (nullable id <UIViewControllerInteractiveTransitioning>)interactionControllerForPresentation:(id <UIViewControllerAnimatedTransitioning>)animator;

- (nullable id <UIViewControllerInteractiveTransitioning>)interactionControllerForDismissal:(id <UIViewControllerAnimatedTransitioning>)animator;

- (nullable UIPresentationController *)presentationControllerForPresentedViewController:(UIViewController *)presented presentingViewController:(UIViewController *)presenting sourceViewController:(UIViewController *)source NS_AVAILABLE_IOS(8_0);

@end
```

前两个方法是针对动画切换的，我们需要分别在呈现ViewController和解散ViewController时，给出一个实现了`UIViewControllerAnimatedTransitioning`接口的对象（其中包含切换时长和如何切换）。后两个方法涉及交互式切换，交互式切换后面会再说，现在先了解这些就够做一些动画了。

##### 交互动画

###### UIViewControllerInteractiveTransitioning

```
@protocol UIViewControllerInteractiveTransitioning <NSObject>
- (void)startInteractiveTransition:(id <UIViewControllerContextTransitioning>)transitionContext;

@optional
- (CGFloat)completionSpeed;
- (UIViewAnimationCurve)completionCurve;

@end
```
