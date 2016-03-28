# 通过LaunchScreen自定义启动动画

大晚上刷微博，刷到一篇转发自[里脊串](http://adad184.com/)的博客[Tips:获取APP的Launch Image](http://adad184.com/2015/10/15/tips-access-current-launch-image/)

大致内容是：通过获取打包到`App`里的启动图，初始化一个`UIImageView`与屏幕同等大小再加载到`UIWindow`上来做启动延时，从而达到自定义启动动画的目的。代码里的各种`Key`没太看懂，放到工程里也没法显示效果。文中的意思是不要增加启动图的方式来适配，那我想了想`iOS`启动画面的方式目前我所知就两种：启动图和布局文件。我现在基本上没有用图片作为启动图，都是直接用布局文件搞定。

PS:今年开发的`App`基本上都是支持`iOS7`以上了，所以没太用启动图了，`Xcode 6`是`LaunchScreen.xib`，到`Xcode 7`变成了`LaunchScreen.storyboard`，无可厚非，两者本质都是一样的。

教程看的云里雾里的，不过不打紧，思路还是一样的，我想如果`Storyboard`和`Size Class`玩得多同学还是喜欢布局文件作为启动图的方式吧，所以我的方式是获取`LaunchScreen.storyboard`里的`ViewController`，在把`View`提取出来加到`UIWindow`显示做动画即可。

这种方式的好处就是，获取大小就是屏幕的大小，而且只要你把不同屏幕的布局搞定了，系统会帮你生成好加在的启动页，这样就免去了判断和从新设置大小的麻烦，这样才是真适配嘛~

废话不多说，上代码吧~
（对了，记得给`LaunchScreen.storyboard`里的`ViewController`设置好`Storyboard ID`）

```objc
UIViewController *viewController = [[UIStoryboard storyboardWithName:@"LaunchScreen" bundle:nil] instantiateViewControllerWithIdentifier:@"LaunchScreen"];

UIView *launchView = viewController.view;
AppDelegate *delegate = [UIApplication sharedApplication].delegate;
UIWindow *mainWindow = delegate.window;
[mainWindow addSubview:launchView];

[UIView animateWithDuration:0.6f delay:0.5f options:UIViewAnimationOptionBeginFromCurrentState animations:^{
    launchView.alpha = 0.0f;
    launchView.layer.transform = CATransform3DScale(CATransform3DIdentity, 1.5f, 1.5f, 1.0f);
} completion:^(BOOL finished) {
    [launchView removeFromSuperview];
}];
```

此代码片段，如果应用启动初始化如果是代码可以在AppDelegate加，Storyboard加载方式需要加在ViewController里。

![](http://upload-images.jianshu.io/upload_images/311224-de90122b4efd3d63.gif?imageMogr2/auto-orient/strip)

---
>2015.11.27
最近项目UI框架切换到`UITabBarController`，发现这个动画没法使用，是由于如果在`Storyboard`中使用`UITabBarController`，如果做启动登录需求，肯定是按需加载，就需要自建继承自`UITabBarController`的关联，如果在`- (void)viewDidLoad`里加载就会导致如下警告：
```
Warning: Attempt to present <HXLoginViewController: 0x7fa5a063cca0> on <HXMainViewController: 0x7fa5a05ae0b0> whose view is not in the window hierarchy!**
```
系统没法知道该怎么显示，所以只能放到`- (void)viewDidAppear:(BOOL)animated`里来做，`UITabBarController`框架加载这个动画没效果也是这个原因，但是没Debug警告，不过要注意，如果只是单纯使用这个动画没啥问题，但是如果在`UITabBarController`上用`模态`视图的方式做按需加载以及转场动画需要处理`- (void)viewDidAppear:(BOOL)animated`重复调用的问题。

---
>2016.03.28
最近看到很多人留言没效果，其实稍微调试一下就知道，不是没效果，是时机出了问题，之前的效果应该在9.1之前能看到，新的系统版本应该改了时机，这里的技巧无非就是在以下三个时机处理：
```objc
- (void)viewDidLoad;
- (void)viewWillAppear:(BOOL)animated;
- (void)viewDidAppear:(BOOL)animated;
```

经过了测试在`viewDidLoad`和`viewWillAppear:`加这个效果都是出现视图层级的问题，这是由于`控制器`生成的`视图`直接盖在了启动页上，层级在其之上当然无法显示。

按照之前的方式在`viewDidLoad`或者`viewWillAppear`实际的效果是这样：
![](http://upload-images.jianshu.io/upload_images/311224-bcbd799350ccbc33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以现在想要看到效果，就只能在最后一个时机`viewDidAppear:`里加入示例代码了。

Demo:[https://github.com/shicang1990/LaunchScreenAnimation-Storyboard](https://github.com/shicang1990/LaunchScreenAnimation-Storyboard)
