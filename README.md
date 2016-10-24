##前言
#####iOS开发过程中一些小问题小技巧，代码片段。
>利用Safari打开一个链接
>
```objc
NSURL *url = [NSURL URLWithString:@"http://baidu.com"];
[[UIApplication sharedApplication] openURL:url];
```
>汉字转码
>
```objc
NSString *oriString = @"\u67aa\u738b";
NSString *escapedString = [oriString stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
```
>调用电话，短信，邮件
>
>```objc
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"mailto:apple@mac.com?Subject=hello"]];
sms://调用短信
tel://调用电话
itms://打开MobileStore.app
```
>获取版本信息
>
>```objc
UIDevice *myDevice = [UIDevice currentDevice];
NSString *systemVersion = myDevice.systemVersion;
```
>
>iPhone 更改键盘右下角按键的 type
>
>```objc
SearchBar *mySearchBar = [[UISearchBar alloc]init];
mySearchBar.frame = CGRectMake(0, 0, self.view.bounds.size.width, 44);
mySearchBar.placeholder = @"placeholderString";
mySearchBar.delegate = self;
[self.view addSubview:mySearchBar];
UITextField *searchField = [[mySearchBar subviews] lastObject];
searchField.returnKeyType = UIReturnKeyDone;
```
>给图片增加模糊效果
>
>```objc
//加模糊效果，image是图片，blur是模糊度
+ (UIImage *)blurryImage:(UIImage *)image withBlurLevel:(CGFloat)blur {
    //模糊度,
    if ((blur < 0.1f) || (blur > 2.0f)) {
        blur = 0.1f;
    }
    //boxSize必须大于0
    int boxSize = (int)(blur * 100);
    boxSize -= (boxSize % 2) + 1;
    NSLog(@"boxSize:%i",boxSize);
    //图像处理
    CGImageRef img = image.CGImage;
>
    //图像缓存,输入缓存，输出缓存
    vImage_Buffer inBuffer, outBuffer;
    vImage_Error error;
    //像素缓存
    void *pixelBuffer;
>
	//数据源提供者，Defines an opaque type that supplies 	Quartz with data.
   CGDataProviderRef inProvider = CGImageGetDataProvider(img);
    // provider’s data.
    CFDataRef inBitmapData = CGDataProviderCopyData(inProvider);
>
	 //宽，高，字节/行，data
    inBuffer.width = CGImageGetWidth(img);
    inBuffer.height = CGImageGetHeight(img);
    inBuffer.rowBytes = CGImageGetBytesPerRow(img);
    inBuffer.data = (void*)CFDataGetBytePtr(inBitmapData);
>
	 //像数缓存，字节行*图片高
    pixelBuffer = malloc(CGImageGetBytesPerRow(img) * CGImageGetHeight(img));
>
    outBuffer.data = pixelBuffer;
    outBuffer.width = CGImageGetWidth(img);
    outBuffer.height = CGImageGetHeight(img);
    outBuffer.rowBytes = CGImageGetBytesPerRow(img);
>
    // 第三个中间的缓存区,抗锯齿的效果
    void *pixelBuffer2 = malloc(CGImageGetBytesPerRow(img) * CGImageGetHeight(img));
    vImage_Buffer outBuffer2;
    outBuffer2.data = pixelBuffer2;
    outBuffer2.width = CGImageGetWidth(img);
    outBuffer2.height = CGImageGetHeight(img);
    outBuffer2.rowBytes = CGImageGetBytesPerRow(img);
>
    //Convolves a region of interest within an ARGB8888 source image by an implicit M x N kernel that has the effect of a box filter.
    error = vImageBoxConvolve_ARGB8888(&inBuffer;, &outBuffer2;, NULL, 0, 0, boxSize, boxSize, NULL, kvImageEdgeExtend);
    error = vImageBoxConvolve_ARGB8888(&outBuffer2;, &inBuffer;, NULL, 0, 0, boxSize, boxSize, NULL, kvImageEdgeExtend);
    error = vImageBoxConvolve_ARGB8888(&inBuffer;, &outBuffer;, NULL, 0, 0, boxSize, boxSize, NULL, kvImageEdgeExtend);
>
    if (error) {
        NSLog(@"error from convolution %ld", error);
    }
>
    //    NSLog(@"字节组成部分：%zu",CGImageGetBitsPerComponent(img));
    //颜色空间DeviceRGB
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
>
    //用图片创建上下文,CGImageGetBitsPerComponent(img),7,8
    CGContextRef ctx = CGBitmapContextCreate(
                                             outBuffer.data,
                                             outBuffer.width,
                                             outBuffer.height,
                                             8,
                                             outBuffer.rowBytes,
                                             colorSpace,
                                             CGImageGetBitmapInfo(image.CGImage));
>
    //根据上下文，处理过的图片，重新组件
    CGImageRef imageRef = CGBitmapContextCreateImage (ctx);
    UIImage *returnImage = [UIImage imageWithCGImage:imageRef];
>
    //clean up
    CGContextRelease(ctx);
    CGColorSpaceRelease(colorSpace);
    free(pixelBuffer);
    free(pixelBuffer2);
    CFRelease(inBitmapData);
    CGImageRelease(imageRef);
>
    return returnImage;
}
```
>图片压缩
>
>```objc
用法：UIImage *yourImage= [self imageWithImageSimple:image scaledToSize:CGSizeMake(210.0, 210.0)];
//压缩图片
- (UIImage*)imageWithImageSimple:(UIImage*)image scaledToSize:(CGSize)newSize
{
// Create a graphics image context
UIGraphicsBeginImageContext(newSize);
// Tell the old image to draw in this newcontext, with the desired
// new size
[image drawInRect:CGRectMake(0,0,newSize.width,newSize.height)];
// Get the new image from the context
UIImage* newImage = UIGraphicsGetImageFromCurrentImageContext();
// End the context
UIGraphicsEndImageContext();
// Return the new image.
return newImage;
}
```
>把时间戳转换为时间
>
>```objc
+ (NSDate *)dateWithTimeIntervalInMilliSecondSince1970:(double)timeIntervalInMilliSecond {
	NSDate *ret = nil;
    double timeInterval = timeIntervalInMilliSecond;
    if(timeIntervalInMilliSecond > 140000000000) {
        timeInterval = timeIntervalInMilliSecond / 1000;
    }
     ret = [NSDate dateWithTimeIntervalSince1970:timeInterval];
>
     return ret;
}
>```
>自定义cell中获取不到cell实际大小的办法
>
>```objc
>-(void)drawRect:(CGRect)rect {
    // 重写此方法，并在此方法中获取
    CGFloat width = self.frame.size.width;
}
>```
>长按图标抖动
>
>```objc
>-(void)longPress:(UILongPressGestureRecognizer*)longPress
{
    if (longPress.state==UIGestureRecognizerStateBegan) {
        CAKeyframeAnimation* anim=[CAKeyframeAnimation animation];
        anim.keyPath=@"transform.rotation";
        anim.values=@[@(angelToRandian(-7)),@(angelToRandian(7)),@(angelToRandian(-7))];
        anim.repeatCount=MAXFLOAT;
        anim.duration=0.2;
        [self.imageView.layer addAnimation:anim forKey:nil];
        self.btn.hidden=NO;
    }
}
>```
>两种方法删除NSUserDefaults所有记录
>
>```objc
>//方法一
NSString *appDomain = [[NSBundle mainBundle] bundleIdentifier];
[[NSUserDefaults standardUserDefaults] removePersistentDomainForName:appDomain];
>
//方法二
- (void)resetDefaults {
    NSUserDefaults * defs = [NSUserDefaults standardUserDefaults];
    NSDictionary * dict = [defs dictionaryRepresentation];
    for (id key in dict) {
        [defs removeObjectForKey:key];
    }
    [defs synchronize];
}
>```
>截屏全图
>
>```objc
>- (UIImage *)imageFromView: (UIView *) theView{
  	UIGraphicsBeginImageContext(theView.frame.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    [theView.layer renderInContext:context];
    UIImage *theImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
>
    return theImage;
}
>```
>去除UIImageView锯齿
>
>```objc
>imageView.layer.shouldRasterize = YES;
>```
>由身份证号码返回性别
>
>```objc
>-(NSString *)sexStrFromIdentityCard:(NSString *)numberStr{
    NSString *result = nil;
    BOOL isAllNumber = YES;
>
    if([numberStr length]<17)
        return result;
>    
    //**截取第17为性别识别符
    NSString *fontNumer = [numberStr substringWithRange:NSMakeRange(16, 1)];
>    
    //**检测是否是数字;
    const char *str = [fontNumer UTF8String];
    const char *p = str;
    while (*p!='\0') {
        if(!(*p>='0'&&*p<='9'))
            isAllNumber = NO;
        p++;
    }
>
    if(!isAllNumber)
        return result;
>
    int sexNumber = [fontNumer integerValue];
    if(sexNumber%2==1)
        result = @"男";
    else if (sexNumber%2==0)
        result = @"女";
>
    return result;  
}
>```
>数组随机重新排列
>
>```objc
>+ (NSArray *)getRandomWithPosition:(NSInteger)position positionContent:(id)positionContent array:(NSArray *)baseArray {
    NSMutableArray *resultArray = [NSMutableArray arrayWithCapacity:baseArray.count];
    NSMutableArray *tempBaseArray = [NSMutableArray arrayWithArray:baseArray];
>
    while ([tempBaseArray count]) {
        NSInteger range = [tempBaseArray count];
        id string = [tempBaseArray objectAtIndex:arc4random()%range];
        [resultArray addObject:string];
        [tempBaseArray removeObject:string];
    }
> 
    NSUInteger index = [resultArray indexOfObject:positionContent];
    [resultArray exchangeObjectAtIndex:index withObjectAtIndex:position - 1];
> 
    return resultArray;
}
>```
>利用陀螺仪实现更真实的微信摇一摇动画
>
>```objc
>- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{    
    application.applicationSupportsShakeToEdit=YES;
}
>
-(void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event {
    if(motion==UIEventSubtypeMotionShake) {
         // 真实一点的摇动动画
        [self addAnimations];
         // 播放声音
        AudioServicesPlaySystemSound (soundID); 
    }
}
>
- (void)addAnimations {
    CABasicAnimation *translation = [CABasicAnimation animationWithKeyPath:@"transform"];
    translation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    translation.toValue=[NSValue valueWithCATransform3D:CATransform3DMakeRotation(-M_PI_4, 0, 0, 100)];
    translation.duration = 0.2;
    translation.repeatCount = 2;
    translation.autoreverses = YES;
>
    [shake.layer addAnimation:translation forKey:@"translation"];
}
>```
>在后台播放音乐
>
>```objc
>//1. 在Info.plist中，添加"Required background modes"键，其值设置是“App plays audio" 
//2. 在播放器播放音乐的代码所在处，添加如下两段代码（当然，前提是已经添加了AVFoundation框架）
>
//添加后台播放代码：
AVAudioSession *session = [AVAudioSession sharedInstance];    
[session setActive:YES error:nil];    
[session setCategory:AVAudioSessionCategoryPlayback error:nil];   
> 
//以及设置app支持接受远程控制事件代码。设置app支持接受远程控制事件，
//其实就是在dock中可以显示应用程序图标，同时点击该图片时，打开app。
//或者锁屏时，双击home键，屏幕上方出现应用程序播放控制按钮。
[[UIApplication sharedApplication] beginReceivingRemoteControlEvents]; 
>
//用下列代码播放音乐，测试后台播放
// 创建播放器  
AVAudioPlayer *player = [[AVAudioPlayer alloc] initWithContentsOfURL:url error:nil];    
[player prepareToPlay];  
[player setVolume:1];  
player.numberOfLoops = -1; //设置音乐播放次数  -1为一直循环  
[player play]; //播放
>```
>利用富文本改变字体颜色
>
>```objc
>- (NSMutableAttributedString *)changeTextColor:(NSString *)text needChange:(NSString *)value UIColor:(UIColor *)color{
>
    NSMutableAttributedString *attrstring = [[NSMutableAttributedString alloc] initWithString:text];
>
    //1.设置字体颜色
    [attrstring addAttribute:NSForegroundColorAttributeName
                       value:color
                       range:[text rangeOfString:value]];
>
    return attrstring;
}
>```
>计算两个日期之间的天数
>
>```objc
>- (NSInteger)calcDaysFromBegin:(NSDate *)beginDate end:(NSDate *)endDate{
    //创建日期格式化对象
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    [dateFormatter setDateFormat:@"yyyy-MM-dd HH:mm"];
>
    //取两个日期对象的时间间隔：
    //这里的NSTimeInterval 并不是对象，是基本型，其实是double类型，是由c定义的:typedef double NSTimeInterval;
    NSTimeInterval time = [endDate timeIntervalSinceDate:beginDate];
>
    int days = ((int)time) / (3600*24);
    //int hours=((int)time)%(3600*24)/3600;
    //NSString *dateContent=[[NSString alloc] initWithFormat:@"%i天%i小时",days,hours];
    return days;
}
>```
>获取上个月或者下个月 +1是下个月 -1是上个月
>
>```objc
>- (NSDate *)getPriousorLaterDateFromDate:(NSDate *)date withMonth:(int)month{
>    
    NSDateComponents *comps = [[NSDateComponents alloc] init];
>    
    [comps setMonth:month];
>   
    NSCalendar *calender = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
>  
    NSDate *mDate = [calender dateByAddingComponents:comps toDate:date options:0];
>   
    return mDate;
>  
}
>```
>两个时间比较
>
>```objc
>- (int)compareOneDay:(NSDate *)oneDay withAnotherDay:(NSDate *)anotherDay{
>
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    [dateFormatter setDateFormat:@"yyyy/MM/dd"];
    NSString *oneDayStr = [dateFormatter stringFromDate:oneDay];
    NSString *anotherDayStr = [dateFormatter stringFromDate:anotherDay];
    NSDate *dateA = [dateFormatter dateFromString:oneDayStr];
    NSDate *dateB = [dateFormatter dateFromString:anotherDayStr];
    NSComparisonResult result = [dateA compare:dateB];
    //    NSLog(@"date1 : %@, date2 : %@", oneDay, anotherDay);
    if (result == NSOrderedDescending) {
        //NSLog(@"Date1  is in the future");
        return 1;
    }
    else if (result == NSOrderedAscending){
        //NSLog(@"Date1 is in the past");
        return -1;
    }
    //NSLog(@"Both dates are the same");
    return 0;
>  
}
>```
>Date 转 String
>
>```objc
- (NSString *)stringFromDate:(NSDate *)date{
>
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
>   
    [dateFormatter setDateFormat:@"yyyy/MM/dd"];
>   
    NSString *destDateString = [dateFormatter stringFromDate:date];
>   
    return destDateString;
>   
}
>```
>String 转Date
>
>```objc
>- (NSDate *)dateFromString:(NSString *)dateString{
>
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
>   
    [dateFormatter setDateFormat: @"yyyy/MM/dd"];
>   
    NSDate *destDate= [dateFormatter dateFromString:dateString];
>   
    return destDate;
>   
}
>```
>浮点转百分比
>
>```objc
- (NSString *)percentage:(NSString *)value{
>
    float _percent = [value floatValue];
>   
    CFLocaleRef currentLocale = CFLocaleCopyCurrent();
>   
    CFNumberFormatterRef numberFormatter = CFNumberFormatterCreate(NULL, currentLocale, kCFNumberFormatterPercentStyle);
>   
    CFNumberRef number = CFNumberCreate(NULL, kCFNumberFloatType, &_percent);
>   
    CFStringRef numberString = CFNumberFormatterCreateStringWithNumber(NULL, numberFormatter, number);
>   
    return (__bridge NSString * _Nullable)(numberString);
>   
}
>```
>获取当月第一天跟最后一天
>
>```objc
- (NSString *)getMonthBeginAndEndWith:(NSString *)dateStr{
>
    NSDateFormatter *format=[[NSDateFormatter alloc] init];
    [format setDateFormat:@"yyyy/MM/dd"];
    NSDate *newDate=[format dateFromString:dateStr];
    double interval = 0;
    NSDate *beginDate = nil;
    NSDate *endDate = nil;
    NSCalendar *calendar = [NSCalendar currentCalendar];
>
    [calendar setFirstWeekday:2];//设定周一为周首日
    BOOL ok = [calendar rangeOfUnit:NSMonthCalendarUnit startDate:&beginDate interval:&interval forDate:newDate];
    //分别修改为 NSDayCalendarUnit NSWeekCalendarUnit NSYearCalendarUnit
    if (ok) {
        endDate = [beginDate dateByAddingTimeInterval:interval-1];
    }else {
        return @"";
    }
    NSDateFormatter *myDateFormatter = [[NSDateFormatter alloc] init];
    [myDateFormatter setDateFormat:@"YYYY/MM/dd"];
    //月初
    NSString *beginString = [myDateFormatter stringFromDate:beginDate];
    //月末
    NSString *endString = [myDateFormatter stringFromDate:endDate];
    NSString *s = [NSString stringWithFormat:@"%@-%@",beginString,endString];
    return s;
}
>```
>UIColor HEX
>
>```objc
>+ (UIColor *)colorWithHexString:(NSString *)color alpha:(CGFloat)alpha{
    //删除字符串中的空格
    NSString *cString = [[color stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]] uppercaseString];
    // String should be 6 or 8 characters
    if ([cString length] < 6)
    {
        return [UIColor clearColor];
    }
    // strip 0X if it appears
    //如果是0x开头的，那么截取字符串，字符串从索引为2的位置开始，一直到末尾
    if ([cString hasPrefix:@"0X"])
    {
        cString = [cString substringFromIndex:2];
    }
    //如果是#开头的，那么截取字符串，字符串从索引为1的位置开始，一直到末尾
    if ([cString hasPrefix:@"#"])
    {
        cString = [cString substringFromIndex:1];
    }
    if ([cString length] != 6)
    {
        return [UIColor clearColor];
    }
>
    // Separate into r, g, b substrings
    NSRange range;
    range.location = 0;
    range.length = 2;
    //r
    NSString *rString = [cString substringWithRange:range];
    //g
    range.location = 2;
    NSString *gString = [cString substringWithRange:range];
    //b
    range.location = 4;
    NSString *bString = [cString substringWithRange:range];
>
    // Scan values
    unsigned int r, g, b;
    [[NSScanner scannerWithString:rString] scanHexInt:&r];
    [[NSScanner scannerWithString:gString] scanHexInt:&g];
    [[NSScanner scannerWithString:bString] scanHexInt:&b];
    return [UIColor colorWithRed:((float)r / 255.0f) green:((float)g / 255.0f) blue:((float)b / 255.0f) alpha:alpha];
}
>```

###Objective-C 内存管理
1. 一个对象可以有一个或多个拥有者
2. 当它一个拥有者都没有的时候，它就会被回收
3. 如果想保留一个对象不被回收，你就必须成为它的拥有者

###关键字
1. alloc 为对象分配内存，计数设为1，并返回此对象。
2. copy 复制一个对象，此对象计数为1，返回此对象。你将成为此克隆对象的拥有者。
3. retain 对象计数+1，并成为次对象的拥有者。
4. release 对象计数-1，并丢掉此对象。
5. autorelease 在未来的
6. 某一个时刻，对象计数-1。并在未来的某个时间放弃此对象。

###原则
1. 一个代码块内要确保copy，alloc 和 retain 的使用数量与 release 和 autorelease 的数量相等。
2. 在使用以 alloc 或 new 开头或包含 copy 的方法，或 retain 一个对象时，你将会编程它的拥有者。
3. 实现 dealloc 方法，这是系统当 retain -> 0 的时候，自动调用的。手动调用会引起 retain count 计数错误（多一次的 release）。