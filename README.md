前言
----------

个人首发：http://segmentfault.com/a/1190000003505115

iOS开发过程中，总有那么一些个小问题让人纠结，它们不会让程序崩溃，但是会让人崩溃。除此之外，还将分享一些细节现在我通过自己的总结以及从其他地方的引用，来总结一下一些常见小问题。

本篇长期更新，多积累，多奉献，同时感谢其中一些文章的作者的整理，感谢！

iOS高级开发实战讲解
----------
这是我在网上搜索到的[iOS高级开发实战讲解][1]，由于原文不是很方便浏览，所以我在这里整理一部分出来，方便查阅，同时谢谢原作者。

这里我不是每一个都收录进来，这里只是放出一部分，有些用的太多，我就没整理了，大家如果想看可以去看原文。

返回输入键盘
```
<UITextFieldDelegate>

-(BOOL)textFieldShouldReturn:(UITextField *)textField {
    [textField resignFirstResponder];
    return YES;
}
```
CGRect
```
CGRectFromString(<#NSString *string#>)//有字符串恢复出矩形
CGRectInset(<#CGRect rect#>, <#CGFloat dx#>, <#CGFloat dy#>)//创建较小或者较大的矩形
CGRectIntersectsRect(<#CGRect rect1#>, <#CGRect rect2#>)//判断两巨星是否交叉，是否重叠
CGRectZero//高度和宽度为零的，位于（0，0）的矩形常量
```
隐藏状态栏
```
[UIApplication sharedApplication] setStatusBarHidden:<#(BOOL)#> withAnimation:<#(UIStatusBarAnimation)#>//隐藏状态栏
```
自动适应父视图大小
```
self.view.autoresizesSubviews = YES;
    self.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
```
UITableView的一些方法
```
这里我自己做了个测试，缩进级别设置为行号，row越大，缩进越多
<UITableViewDelegate>

- (NSInteger)tableView:(UITableView *)tableView indentationLevelForRowAtIndexPath:(NSIndexPath *)indexPath {
    NSInteger row = indexPath.row;
    return row;
}
```
把plist文件中的数据赋给数组

```
NSString *path = [[NSBundle mainBundle] pathForResource:@"States" ofType:@"plist"];
NSArray *array = [NSArray arrayWithContentsOfFile:path];
```
获取触摸的点

```
- (CGPoint)locationInView:(UIView *)view;
- (CGPoint)previousLocationInView:(UIView *)view;
```
获取触摸的属性

```
@property(nonatomic,readonly) NSTimeInterval      timestamp;
@property(nonatomic,readonly) UITouchPhase        phase;
@property(nonatomic,readonly) NSUInteger          tapCount;
```
从plist中获取数据赋给字典

```
NSString *plistPath = [[NSBundle mainBundle] pathForResource:@"book" ofType:@"plist"];
NSDictionary *dictionary = [NSDictionary dictionaryWithContentsOfFile:plistPath];
```
NSUserDefaults注意事项

```
设置完了以后如果存储的东西比较重要的话，一定要同步一下
[[NSUserDefaults standardUserDefaults] synchronize];
```

获取Documents目录

```
NSString *documentsDirectory = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
```
获取tmp目录

```
NSString *tmpPath = NSTemporaryDirectory();
```
利用Safari打开一个链接

```
NSURL *url = [NSURL URLWithString:@"http://baidu.com"];
[[UIApplication sharedApplication] openURL:url];
```
利用UIWebView显示pdf文件，网页等等

```
<UIWebViewDelegate>

UIWebView *webView = [[UIWebView alloc]initWithFrame:self.view.bounds];
webView.delegate = self;
webView.scalesPageToFit = YES;
webView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
[webView setAllowsInlineMediaPlayback:YES];
[self.view addSubview:webView];
    
NSString *pdfPath = [[NSBundle mainBundle] pathForResource:@"book" ofType:@"pdf"];
NSURL *url = [NSURL fileURLWithPath:pdfPath];
NSURLRequest *request = [NSURLRequest requestWithURL:url cachePolicy:(NSURLRequestUseProtocolCachePolicy) timeoutInterval:5];
[webView loadRequest:request];
```
UIWebView和html的简单交互

```
myWebView = [[UIWebView alloc]initWithFrame:self.view.bounds];
[myWebView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]]];
NSError *error;
NSString *errorString = [NSString stringWithFormat:@"<html><center><font size=+5 color='red'>AnError Occurred;<br>%@</font></center></html>",error];
[myWebView loadHTMLString:errorString baseURL:nil];

//页面跳转了以后，停止载入
-(void)viewWillDisappear:(BOOL)animated {
    if (myWebView.isLoading) {
        [myWebView stopLoading];
    }
    myWebView.delegate = nil;
    [UIApplication sharedApplication].networkActivityIndicatorVisible = NO;
}
```
汉字转码

```
NSString *oriString = @"\u67aa\u738b";
NSString *escapedString = [oriString stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
```
处理键盘通知
```
先注册通知，然后实现具体当键盘弹出来要做什么，键盘收起来要做什么
- (void)registerForKeyboardNotifications {
    keyboardShown = NO;//标记当前键盘是没有显示的
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWasShown:) name:UIKeyboardWillShowNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWasHidden:) name:UIKeyboardDidHideNotification object:nil];
}
//键盘显示要做什么
- (void)keyboardWasShown:(NSNotification *)notification {
    if (keyboardShown) {
        return;
    }
    NSDictionary *info = [notification userInfo];
    
    NSValue *aValue = [info objectForKey:UIKeyboardFrameBeginUserInfoKey];
    CGSize keyboardSize = [aValue CGRectValue].size;
    CGRect viewFrame = scrollView.frame;
    viewFrame.size.height = keyboardSize.height;
    
    CGRect textFieldRect = activeField.frame;
    [scrollView scrollRectToVisible:textFieldRect animated:YES];
    keyboardShown = YES;
}

- (void)keyboardWasHidden:(NSNotification *)notification {
    NSDictionary *info = [notification userInfo];
    NSValue *aValue = [info objectForKey:UIKeyboardFrameEndUserInfoKey];
    CGSize keyboardSize = [aValue CGRectValue].size;
    
    CGRect viewFrame = scrollView.frame;
    viewFrame.size.height += keyboardSize.height;
    scrollView.frame = viewFrame;
    
    keyboardShown = NO;
}
```
点击键盘的next按钮，在不同的textField之间换行

```
- (BOOL)textFieldShouldReturn:(UITextField *)textField {
    
    if ([textField returnKeyType] != UIReturnKeyDone) {
        NSInteger nextTag = [textField tag] + 1;
        UIView *nextTextField = [self.tableView viewWithTag:nextTag];
        [nextTextField becomeFirstResponder];
    }else {
        [textField resignFirstResponder];
    }
    
    return YES;
}
```

总结
----------
不积跬步无以至千里，不积小流无以成江海。
（随时更新）

  [1]: http://download.csdn.net/detail/spritekit/8426869
