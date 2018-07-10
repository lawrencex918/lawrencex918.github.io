---
published: true
tags: iOS
title: iOS中图片解码、缓存的时机
date: 'Tue Jul 10 2018 14:36:43 GMT+0800'
---
## iOS图片解码的时机
通过 imageNamed 创建 UIImage 时，系统实际上只是在 Bundle 内查找到文件名，然后把这个文件名放到 UIImage 里返回，并没有进行实际的文件读取和解码。当 UIImage 第一次显示到屏幕上时，其内部的解码方法才会被调用，同时解码结果会保存到一个全局缓存去。据我观察，在图片解码后，App 第一次退到后台和收到内存警告时，该图片的缓存才会被清空，其他情况下缓存会一直存在。

## 最准确的结论：
手动调用CGImageSourceCreateWithData自己绘制图片，将图片绘制到一个Bitmap上，这个过程是立即解码的，而且是线程安全的。
其他UIImage这一层的接口，都是**线程不安全**的（iOS9以后被官方修改成线程安全的了），而且不管是imageNamed还是initWithData，其实都是在图片被首次加载到页面上的时候，才会被解码的。
图片的解码可以放后台线程，渲染不能。

### 关于Cache
其实针对图片解码和缓存的逻辑，都可以从ImageIO里面找到解答。CGImageSource.h文件里面的注释已经说的很明白了，在生成UIImage的时候，有两个重要的枚举量控制了缓存的时机。

```objc
/* Specifies whether the image should be cached in a decoded form. The
 * value of this key must be a CFBooleanRef.
 * kCFBooleanFalse indicates no caching, kCFBooleanTrue indicates caching.
 * For 64-bit architectures, the default is kCFBooleanTrue, for 32-bit the default is kCFBooleanFalse.
 */

IMAGEIO_EXTERN const CFStringRef kCGImageSourceShouldCache;

/* Specifies whether image decoding and caching should happen at image creation time.
 * The value of this key must be a CFBooleanRef. The default value is kCFBooleanFalse (image decoding will
 * happen at rendering time).
 */
IMAGEIO_EXTERN const CFStringRef kCGImageSourceShouldCacheImmediately;
```
其中kCGImageSourceShouldCacheImmediately这个枚举默认是false，会导致图片的解码和缓存都是在图片第一次被渲染的时候才执行的。
kCGImageSourceShouldCache这个枚举在64位架构（截止到目前市面上的主流iPhone机型）的机器上默认是true。导致其实生成UIImage的时候，解码完成的时候都会缓存。

### 解释下imageName和initWithContentOfFile的缓存之谜
网上有些博客说imageName是缓存图片，initWithContentOfFile不会缓存图片。其实不然，initWithContentOfFile其实也是缓存了图片的，不同之处在于这个方法在缓存图片的时候会给图片打一个可消除的标记，保证对应的图片资源被释放的时候，对应的缓存也会被释放。而imageName这种方式，只会在APP收到内存警告的时候进行释放。这才是两者真正的不同。

引用下官方文档对于initWithContentOfFile方法的Discussion
> This method loads the image data into memory and marks it as purgeable. If the data is purged and needs to be reloaded, the image object loads that data again from the specified path.


## 参考文章
网上很多文章对解码和缓存的解释都不是很明确，而且各种重复的转载，还好有一些干货文章。
[iOS处理图片的一些小Tips](https://blog.ibireme.com/2015/11/02/ios_image_tips/)

[iOS中的imageIO与image解码](http://ios.jobbole.com/87233/)
