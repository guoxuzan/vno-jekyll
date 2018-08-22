---
layout: post
title: UITableViewCell多级折叠/树形列表
---
### 先看效果
![](https://github.com/guoxuzan/vno-jekyll/blob/master/assets/2017071601.gif)
### 源码
[https://github.com/guoxuzan/ZZFoldCell](https://github.com/guoxuzan/ZZFoldCell){:target="_blank"}
### 原理分析
图中的行政区域是树形结构，但UITableView的Cell是队列形式，展开与折叠效果靠UITableView的Cell插入删除实现。
```
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
[tableView beginUpdates];
if (//要展开) {
//子节点成为兄弟节点

//insertRows
[tableView insertRowsAtIndexPaths:indexPaths withRowAnimation:UITableViewRowAnimationFade];
}else {//要折叠
//兄弟节点变回子节点

//deleteRows
[tableView deleteRowsAtIndexPaths:indexPaths withRowAnimation:UITableViewRowAnimationFade];
}
[tableView endUpdates];
}
```
树的操作靠Model配合data实现
```
@interface ZZFoldCellModel : NSObject

@property(nonatomic,copy) NSString *text;//cell展示内容
@property(nonatomic,copy) NSString *level;//节点级别
//...

@property(nonatomic,assign) NSUInteger belowCount;//子节点已经转为兄弟节点的个数
@property(nullable,nonatomic) ZZFoldCellModel *supermodel;//父节点
@property(nonatomic,strong) NSMutableArray<__kindof ZZFoldCellModel *> *submodels;//子节点

+ (instancetype)modelWithDic:(NSDictionary *)dic;//初始化
- (NSArray *)open;//返回并删除子节点
- (void)closeWithSubmodels:(NSArray *)submodels;//插入子节点

@end
```
### 具体实现
```
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
ZZFoldCellModel *didSelectFoldCellModel = self.data[indexPath.row];

[tableView beginUpdates];
if (didSelectFoldCellModel.belowCount == 0) {

//Data
NSArray *submodels = [didSelectFoldCellModel open];
NSIndexSet *indexes = [NSIndexSet indexSetWithIndexesInRange:((NSRange){indexPath.row + 1,submodels.count})];
[self.data insertObjects:submodels atIndexes:indexes];

//Rows
NSMutableArray *indexPaths = [NSMutableArray new];
for (int i = 0; i < submodels.count; i++) {
NSIndexPath *insertIndexPath = [NSIndexPath indexPathForRow:(indexPath.row + 1 + i) inSection:indexPath.section];
[indexPaths addObject:insertIndexPath];
}
[tableView insertRowsAtIndexPaths:indexPaths withRowAnimation:UITableViewRowAnimationFade];

}else {

//Data
NSArray *submodels = [self.data subarrayWithRange:((NSRange){indexPath.row + 1,didSelectFoldCellModel.belowCount})];
[didSelectFoldCellModel closeWithSubmodels:submodels];
[self.data removeObjectsInArray:submodels];

//Rows
NSMutableArray *indexPaths = [NSMutableArray new];
for (int i = 0; i < submodels.count; i++) {
NSIndexPath *insertIndexPath = [NSIndexPath indexPathForRow:(indexPath.row + 1 + i) inSection:indexPath.section];
[indexPaths addObject:insertIndexPath];
}
[tableView deleteRowsAtIndexPaths:indexPaths withRowAnimation:UITableViewRowAnimationFade];
}
[tableView endUpdates];
}
```
