---
title: Flutter 拖动排序列表与跨平台优化实践
date: 2021-03-30
lastmod: 2021-03-30
description: Flutter 拖动排序列表与跨平台优化实践
categories:
- 技术
tags:
- 技术
- Flutter
---

<!-- # Flutter 拖动排序列表与跨平台优化实践 -->



Flutter 中实现拖动排序的列表非常简单，使用官方的`ReorderableListView`替代原本的`ListView`即可，`ListView.builder`同理。



```dart
  return ReorderableListView(
    // padding: const EdgeInsets.symmetric(horizontal: 40),
    children: <Widget>[
      for (int index = 0; index < _items.length; index++)
        ListTile(
          key: Key('$index'),
          tileColor: _items[index].isOdd ? oddItemColor : evenItemColor,
          title: Text('Item ${_items[index]}'),
        ),
    ],
    onReorder: (int oldIndex, int newIndex) {
      setState(() {
        if (oldIndex < newIndex) {
          newIndex -= 1;
        }
        final int item = _items.removeAt(oldIndex);
        _items.insert(newIndex, item);
      });
    },
  );
}
```

上面是官方给的 demo，简洁明了。[官方介绍视频](https://www.youtube.com/watch?v=yll3SNXvQCw)下面的评论里，人家直呼比原生 Android 写的过瘾的多。阅读文档后我给我的 App 的界面也加上了可拖动排序的功能。效果如下图



![截屏 2021-03-30 23.04.59](https://blog-1301127393.file.myqcloud.com/BlogImgs/20210330234001.png)

虽然可以实现拖动了，但是右边有一个按钮很碍眼。不过这个是用来控制拖动的，鼠标移上去才能触发拖动。

翻了翻文档发现，这个地方`ReorderableListView`在移动端和桌面端的处理是不一样的，上图（桌面 macos）右边会出现一个 dragHandler，而移动端则是靠长按列表项触发拖动。

要把这个碍眼的图标去掉，有两个方案：

- 找到定义这个图标的地方，更换一个合适的图标，并在外面包裹一层，仅在鼠标 hover 时将图标显示以进行拖动，其他情况下隐藏
- 将拖动的实现定义为和移动端相同，即都通过长按列表项触发拖动



针对方案一，以`ReorderableListView` 和`Icon`为关键词搜索，很遗憾的是并没有这方面的资料，又翻了翻源码，并没有找到可以自定义右边这个拖拽按钮的实现。

在 Flutter 官方的 Github 仓库的 issue 中，[https://github.com/flutter/flutter/issues/66080#issuecomment-771123430](https://github.com/flutter/flutter/issues/66080#issuecomment-771123430)   对相关问题做了阐述，并有提案对`ReorderableListView`进行了改进。该 issue 对应的 pr 已经被合并到 stable channel，查阅相关 API 后了解到可以使用`buildDefaultDragHandles: false`，关闭默认的拖拽触发实现，再使用`ReorderableDelayedDragStartListener`包裹原先的`ListTile`即可实现桌面端和移动端都通过长按列表项触发拖拽排序。

不过相应的，ListTile 的`onLongPress`就不能再有响应了。刚好今天完成了滑动删除的实现，现在列表也的删除、排序已经高度可用且多平台统一了。

![截屏 2021-03-30 23.21.32](https://blog-1301127393.file.myqcloud.com/BlogImgs/20210330233350.png)

附相关代码实现：



```dart
body: ReorderableListView.builder(
  buildDefaultDragHandles: false,
  onReorder: (int oldIndex, int newIndex) {
    setState(() {
      if (oldIndex < newIndex) newIndex -= 1;
      var temp = l.removeAt(oldIndex);
      l.insert(newIndex, temp);
    });
  },
  itemCount: l.length,
  itemBuilder: (context, index) {
    final item = l[index];
    return Dismissible(
        key: item.key,
        background: listTileBackground(),
        onDismissed: (direction) {
          setState(() {
            lastDeleted = item;
            l.removeAt(index);
          });
          ScaffoldMessenger.of(context).showSnackBar(msgDeleted);
        },
        child: ReorderableDelayedDragStartListener(
            key: item.key,
            index: index,
            child: ListTile(
              title: Text(item.title),
              subtitle: Text(item.content),
              contentPadding: EdgeInsets.fromLTRB(16, 8, 16, 8),
              onTap: () {
                //your function here
              },
              // onLongPress: () {
              //   lastDeleted = item;
              //   l.removeAt(index);
              //   setState(() {});
              //   ScaffoldMessenger.of(context)
              //       .showSnackBar(msgDeleted);
              // },
            )));
  })));
```

