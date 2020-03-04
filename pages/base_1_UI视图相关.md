## UI视图相关问题
- UITableView
> 主要涉及到重用机制以及应用方面
```
原理： 重用机制主要用到了一个可变数组`visiableCells`和一个可变的字典类型`reusableTableCells`，其中`visiableCells`用来
存储当前UITableView显示的cell，`reusableTableCells`用来存储已经用`identity`缓存的cell。当UITableView滚动的时候，会先
在`reusableTableCells`中根据`identify`找是否只有已经缓存的cell，如果有直接使用，没有则初始化一个。
```
