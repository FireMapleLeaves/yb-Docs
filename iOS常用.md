##### OC排序

- NSOrderedAscending：对比之后顺序排

- NSOrderedDescending：对比之后逆序排

例如，降序写法

```objective-c
//a > b 时，将a和b顺序排，即降序
if (a > b) {
	return NSOrderedAscending;
}
//a < b 时，将a和b逆序排，也是降序
if (a < b) {
	return NSOrderedDescending;
}
return NSOrderedSame;
```

```objective-c
  NSArray *messageList = @[@{@"sendTime":@"1.3"},@{@"sendTime":@"1.2"},@{@"sendTime":@"1.5"},@{@"sendTime":@"1.4"}];
  messageList = [messageList sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        if ([[obj1 valueForKey:@"sendTime"] floatValue] > [[obj2 valueForKey:@"sendTime"] floatValue]) {
            return NSOrderedAscending;

        } else if ([[obj1 valueForKey:@"sendTime"] floatValue] < [[obj2 valueForKey:@"sendTime"] floatValue]) {
            return NSOrderedDescending;
        }
        return NSOrderedSame;
    }];
    NSLog(@"排序结果-----%@",[messageList mj_JSONString]);

//输出 排序结果-----[{"sendTime":"1.5"},{"sendTime":"1.4"},{"sendTime":"1.3"},{"sendTime":"1.2"}]

    
```

另一种方法

```objc
//方法二    
NSMutableArray *messageList = [@[@{@"sendTime":@"1.3"},@{@"sendTime":@"1.2"},@{@"sendTime":@"1.5"},@{@"sendTime":@"1.4"}] mutableCopy];
    NSSortDescriptor *sort = [NSSortDescriptor sortDescriptorWithKey:@"sendTime" ascending:NO];  //是否升序
    //给数组添加排序规则
    [messageList sortUsingDescriptors:@[sort]]; //这个NSSortDescriptor可以添加多个，先按第一个排，有相同的再按后面的排
    NSLog(@"排序结果-----%@",[messageList mj_JSONString]);
//输出 排序结果-----[{"sendTime":"1.5"},{"sendTime":"1.4"},{"sendTime":"1.3"},{"sendTime":"1.2"}]
```



