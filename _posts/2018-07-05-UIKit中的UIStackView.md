---
published: true
tags: iOS
date: 'Thu Jul 05 2018 21:34:17 GMT+0800'
---
> UIStackView是从iOS9开始支持的一种Container View，用于解决弹性布局问题。

首先需要明确的是UIStackView本身是一种Container View，这导致它并不会渲染到界面上，对stackview本身设置背景和drawrect属性也是没有意义的。

stackView的优势在于他继承了一部分autolayout功能，能够自动得为放进stackview里面subview添加约束，保证自适应stackView本身的大小。stackView的subview的排列方法有水平和垂直两种方法。

## stackView配置
stackview本身最重要的属性为
![img](http://cc.cocimg.com/api/uploads/20150623/1435027330575371.png)
其中Axis指subview按照什么方向排列(水平还是垂直)，Alignment指的是给予什么对其，Distribution值得是拉伸的策略，Spacing指的每个subview之间间隔的间距。

## subView 和 arrangedSubView
这两者的区别是stackView里面的重点。如果你想添加一个subview给Stack View管理，你应该调用addArrangedSubview:或insertArrangedSubview:atIndex: arrangedSubviews数组是subviews属性的子集。

要移除Stack View管理的subview，需要调用removeArrangedSubview:和removeFromSuperview。移除arrangedSubview只是确保Stack View不再管理其约束，而非从视图层次结构中删除，理解这一点非常重要。


### 针对图片类型的组件
stackView可能会自动拉伸图片，因此在设置imageView性质的subview时需要留意：
``` swift
@IBAction func addStar(sender: AnyObject) {
    let starImgVw:UIImageView = UIImageView(image: UIImage(named: "star"))
    //配置下ImageView的contentmode
    starImgVw.contentMode = .ScaleAspectFit
    self.horizontalStackView.addArrangedSubview(starImgVw)
    UIView.animateWithDuration(0.25, animations: {
        self.horizontalStackView.layoutIfNeeded()
    })
}
```
