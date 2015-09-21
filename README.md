### Encapsulates a return key--封装一个返回键
- 首先，每个APP中基本都会用到这样子的返回按钮，而且，用到的地方太多了

![](http://upload-images.jianshu.io/upload_images/739863-7cf898eea78b784f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](http://upload-images.jianshu.io/upload_images/739863-1e3286908195a686.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 不同的导航控制器之间的切换总会有返回键，而我们不可能，每次有要进行切换的地方就写一个返回键，这样的话代码量太大
- 所以，在这里我统一自定义一个返回按钮，统一设置它的为一种样式的返回键
  + 也就是所有的返回按钮都是一样的，所以不用纠结耦合性问题了
  + 如果项目有有别的需求，那就自定义新的按钮吧
  + 只是一般项目的控制器都是有层级关系，不同层级如果要定义不同的按钮样式。同样根据条件判断层级，再写按钮。
  + 当然这里设置所有的返回按钮都是一样的，但也是有很大的用处的。
- 所以我将它进行了方法的封装，今后使用到返回键的时候只要调用方法就行了。
- 步骤：
  + 首先，自定义一个导航控制器
  + 然后，在导航控制器中重写push：方法

- **注意思考一下：我们第一个控制器是不需要有返回键以及右划手势的**

```objc
#import "CYNavigationController.h"

@interface CYNavigationController () <UIGestureRecognizerDelegate>

@end

@implementation CYNavigationController

- (void)viewDidLoad {
    [super viewDidLoad];
    // 这里设置边界的右划手势
    //设置手势代理
    self.interactivePopGestureRecognizer.delegate = self;
}
// 重写push方法
/**
 *  拦截push进来的所有子控件
 *  @param pushViewController每一次push进来的子控件
 */
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated
{
//    NSLog(@"%s, %@",__func__,viewController);
    // 左上角的返回键
    // 注意：第一个控制器不需要返回键
    // if不是第一个push进来的子控制器{
    if (self.childViewControllers.count >= 1) {
        // 左上角的返回按钮
    UIButton *backButton = [UIButton buttonWithType:UIButtonTypeCustom];
    [backButton setTitle:@"返回" forState:UIControlStateNormal];
    [backButton setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
    [backButton setTitleColor:[UIColor redColor] forState:UIControlStateHighlighted];
    [backButton setImage:[UIImage imageNamed:@"navigationButtonReturn"] forState:UIControlStateNormal];
    [backButton setImage:[UIImage imageNamed:@"navigationButtonReturnClick"] forState:UIControlStateHighlighted];
    [backButton sizeToFit];
    [backButton addTarget:self action:@selector(back) forControlEvents:UIControlEventTouchUpInside];

    backButton.contentEdgeInsets = UIEdgeInsetsMake(0, -20, 0, 0); // 这里微调返回键的位置
    viewController.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc] initWithCustomView:backButton];
        viewController.hidesBottomBarWhenPushed = YES; // 隐藏底部的工具条
}
    // super的push方法一定要写到最后面
    // 一旦调用super的pushViewController方法,就会创建子控制器viewController的view
    // 也就会调用viewController的viewDidLoad方法

    [super pushViewController:viewController animated:animated];

}

- (void)back
{
    [self popViewControllerAnimated:YES];// 这里不用写self.navigationController，因为它自己就是导航控制器
}
- (BOOL)gestureRecognizerShouldBegin:(nonnull UIGestureRecognizer *)gestureRecognizer
{
    // 这里设置边界的右划手势
    // 如果当前显示的是第一个子控制器,就应该禁止掉[返回手势]
    //    if (self.childViewControllers.count == 1) return NO;
    //    return YES;
    return self.childViewControllers.count > 1;
}

@end
```
- 这样就OK了，今后不管你要跳到哪儿去，哪怕里面几十层，你也不用再写返回键
- 今后每次不管是在哪个控制器之间切换，直接push就行
- 例如：下面在主界面，跳转到设置界面里。很轻松就这样实现了



![](http://upload-images.jianshu.io/upload_images/739863-59c9e9f4c0e2f53a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 就算设置界面里还要跳转到别的界面，也不用再写返回键代码。同理别的控制器之间也是如此！

```objc
#import "CYMeViewController.h"
#import "CYSettingViewController.h"

@interface CYMeViewController ()

@end

@implementation CYMeViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    self.navigationItem.title = @"我的";

    // 我的导航栏右边的内容
    UIBarButtonItem *moonButton = [UIBarButtonItem itemWithImage:@"mine-moon-icon" highImage:@"mine-moon-icon-click" target:self action:@selector(moonClick)];
    UIBarButtonItem *settingButton = [UIBarButtonItem itemWithImage:@"mine-setting-icon" highImage:@"mine-setting-icon-click" target:self action:@selector(settingClick)];

    self.navigationItem.rightBarButtonItems = @[settingButton,moonButton];
}
- (void)moonClick
{
    CYLogFunc
}


- (void)settingClick
{
    //    self.navigationController.navigationBar.barTintColor = [UIColor darkGrayColor]; //设置导航栏的颜色
    //    self.navigationController.navigationBar.tintColor = [UIColor yellowColor]; // 设置返回按钮字体的颜色

    CYSettingViewController *setting = [[CYSettingViewController alloc] init];
    [self.navigationController pushViewController:setting animated:YES];
}
```

- 完毕
- 所以这里我们就封装来了一个返回键
- 基本没有代码量，一劳永逸

