---
published: true
title: UITextView响应URL点击自定义
date: 'Thu Jul 12 2018 17:39:34 GMT+0800'
tags: iOS
---
# textView响应点击URL事件
根源问题是这样的，如果只是使用默认的textView捕获代理点击URL的方法的话
```objc
- (BOOL)textView:(UITextView *)textView shouldInteractWithURL:(NSURL *)URL inRange:(NSRange)characterRange {
//    [self tapLink:URL];
    NSLog(@"选中的链接%@", URL.absoluteString);
    return NO;
}
```
发现点击区域会扩大，一般会覆盖链接本身所在行和下一行。
类似这个[链接](https://www.oschina.net/question/872288_2150944)中提到的问题。现象就是点击区域被扩大了。

从网上找到几个解决方案，有基于[Label](https://github.com/Jitu1990/link-clickable-Label)的，也有基于[textView](https://github.com/xiaoaihhh/problemsOfiOS/tree/master/SFTextLinkView/SFTextLinkView)的，但是核心思路都是一样的。这里总结一下解决的思路。

1. 先检测出text中所有URL的range。
2. 拦截掉原来textView的响应，添加手势或者重写touchBegin，拿到点击的point坐标，使用textview.layoutManager.characterIndex方法拿到触摸点的字符在text中对应的range下标index。
3. 在响应函数中遍历第1步中检测出来的URL range数组，用NSLocationInRange检查点击的字符index是否处于URL的range范围，以及处于哪个URL的range范围，根据这个range拿到对应的url内容，继续处理逻辑。



**补充**：检查URL常用的正则：
```
(https?|ftp|file)://[-A-Za-z0-9+&@#/%?=~_|!:,.;]+[-A-Za-z0-9+&@#/%=~_|])
```
