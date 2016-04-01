---
layout:     keynote
title:      "CocoaPods"
date:       2015-09-15
author:     "GoTT"
header-img: "img/post-bg-2015.jpg"
category : [CocoaPods,Tools]
tags: iOS
---
# CocoaPods

## Install CocoaPods
CocoaPods is built with Ruby and is installable with the default Ruby available on OS X. We recommend you do this.

Using the default Ruby install will require you to use sudo when installing gems. Further instructions are in the guides.

```
$ sudo gem install cocoapods
```

## Get started
Search for pods (above). Then list the dependencies in a text file named Podfile in your Xcode project directory:

```
platform :ios, '8.0'
use_frameworks!

target 'MyApp' do
  pod 'AFNetworking', '~> 2.5'
  pod 'ORStackView', '~> 2.0'
  pod 'SwiftyJSON', '~> 2.1'
end
```

Tip: CocoaPods provides a pod init command to create a Podfile with smart defaults. You should use it.

Now you can install the dependencies in your project:

```
$ pod install
```

最近可能由于出国节点的问题，无论是执行pod install还是pod update都卡在Analyzing dependencies不动了，慢到无以复加的地步，无法忍受。其实原因在于以上两个命令执行时会升级CocoaPods的spec仓库，加一个参数可以省略这一步，然后速度就会提升不少。加参数的命令如下：

```
pod install --verbose --no-repo-update
pod update --verbose --no-repo-update
```
