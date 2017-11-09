# UIButtonMutablieClick
UIButton重复点击解决办法

避免一个button被多次点击（共总结了3种）

第一种：每次在点击时先取消之前的操作

将这段代码放在你按钮点击的方法中，例如：
```
- (void)buttonClicked:(id)sender{
//点击按钮后先取消之前的操作，再进行需要进行的操作
[[selfclass]cancelPreviousPerformRequestsWithTarget:selfselector:@selector(buttonClicked:)object:sender];
[selfperformSelector:@selector(buttonClicked: )withObject:senderafterDelay:0.2f];
}
```
第二种：点击后设为不可被点击的状态，几秒后恢复:
```
-(void)buttonClicked:(id)sender{
self.button.enabled =NO;
[selfperformSelector:@selector(changeButtonStatus)withObject:nilafterDelay:1.0f];//防止重复点击
}
-(void)changeButtonStatus{
self.button.enabled =YES;
}
```
第三种：使用runtime，一劳永逸我这设的是0.5秒内不会被重复点击

1.导入objc / runtime.h（可以放在PCH文件里）

2.创建uicontrol或UIButton的的分类！

创建分类文件：

2.1 打开Xcode中，新建文件，选择OC文件

2.2 在第二个界面，File名为UIControl+UIControl_buttonCon，将文件类型File Type选为Category类，在类里选继承的类别，这里咱们选的Class是UIButton

注：若用Unbutton分类，则会对对Unbutton创建的按钮反应。

2.3 分类创建完毕对分类进行操作

.h文件
```
#import
#define defaultInterval.5//默认时间间隔
@interfaceUIControl (UIControl_buttonCon)
@property(nonatomic,assign)NSTimeIntervaltimeInterval;//用这个给重复点击加间隔
@property(nonatomic,assign)BOOLisIgnoreEvent;//YES不允许点击NO允许点击
@end
```
.m文件
```
#import"UIControl+UIControl_buttonCon.h"
@implementationUIControl (UIControl_buttonCon)
- (NSTimeInterval)timeInterval{
return[objc_getAssociatedObject(self,_cmd)doubleValue];
}
- (void)setTimeInterval:(NSTimeInterval)timeInterval{
objc_setAssociatedObject(self,@selector(timeInterval),@(timeInterval),OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
//runtime动态绑定属性
- (void)setIsIgnoreEvent:(BOOL)isIgnoreEvent{
objc_setAssociatedObject(self,@selector(isIgnoreEvent),@(isIgnoreEvent),OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
- (BOOL)isIgnoreEvent{
return[objc_getAssociatedObject(self,_cmd)boolValue];
}
- (void)resetState{
[selfsetIsIgnoreEvent:NO];
}
+ (void)load{
staticdispatch_once_tonceToken;
dispatch_once(&onceToken, ^{
SELselA =@selector(sendAction:to:forEvent:);
SELselB =@selector(mySendAction:to:forEvent:);
MethodmethodA =class_getInstanceMethod(self, selA);
MethodmethodB =class_getInstanceMethod(self, selB);
//将methodB的实现添加到系统方法中也就是说将methodA方法指针添加成方法methodB的返回值表示是否添加成功
BOOLisAdd =class_addMethod(self, selA,method_getImplementation(methodB),method_getTypeEncoding(methodB));
//添加成功了说明本类中不存在methodB所以此时必须将方法b的实现指针换成方法A的，否则b方法将没有实现。
if(isAdd) {
class_replaceMethod(self, selB,method_getImplementation(methodA),method_getTypeEncoding(methodA));
}else{
//添加失败了说明本类中有methodB的实现，此时只需要将methodA和methodB的IMP互换一下即可。
method_exchangeImplementations(methodA, methodB);
}
});
}
- (void)mySendAction:(SEL)action to:(id)target forEvent:(UIEvent*)event{
if([NSStringFromClass(self.class)isEqualToString:@"UIButton"]) {
self.timeInterval=self.timeInterval==0?defaultInterval:self.timeInterval;
if(self.isIgnoreEvent){
return;
}elseif(self.timeInterval>0){
[selfperformSelector:@selector(resetState)withObject:nilafterDelay:self.timeInterval];
}
}
//此处methodA和methodB方法IMP互换了，实际上执行sendAction；所以不会死循环
self.isIgnoreEvent=YES;
[selfmySendAction:actionto:targetforEvent:event];
}
@end
```
