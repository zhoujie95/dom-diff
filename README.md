dom-diff 
=================

[![build](https://circleci.com/gh/livoras/list-diff/tree/master.png?style=shield)](https://circleci.com/gh/livoras/list-diff) [![codecov.io](https://codecov.io/github/livoras/list-diff/coverage.svg?branch=master)](https://codecov.io/github/livoras/list-diff?branch=master) [![npm version](https://badge.fury.io/js/list-diff2.svg)](https://badge.fury.io/js/list-diff2) [![Dependency Status](https://david-dm.org/livoras/list-diff.svg)](https://david-dm.org/livoras/list-diff) 

[![js-standard-style](https://cdn.rawgit.com/feross/standard/master/badge.svg)](https://github.com/feross/standard)


### 一、DOM Diff算法简析
![blockchain](https://img-blog.csdn.net/20180717182348969 )
### 二、DIFF算法三个维度
**Tree DIFF、Component DIFF和Element DIFF**
**执行时按顺序依次执行，它们的差异仅仅因为DIFF粒度不同、执行先后顺序不同。**
1. Tree DIFF
![Tree DIFF](https://img-blog.csdn.net/20180403132519897 )
每一层进行遍历，如果某组件不存在了，则会直接销毁。如图所示，左边是旧属，右边是新属，第一层是R组件，一模一样，不会发生变化；第二层进入Component DIFF，同一类型组件继续比较下去，发现A组件没有，所以直接删掉A、B、C组件；继续第三层，重新创建A、B、C组件。
2. Component DIFF
![Component DIFF](https://img-blog.csdn.net/2018040313254025 )
第一层遍历完，进行第二层遍历时，D和G组件是不同类型的组件，不同类型组件直接进行替换，将D删掉，再将G重建。
3. Element DIFF
![Element DIFF](https://img-blog.csdn.net/20180403132558624 )
一般diff在比较集合[A,B,C,D]和[B，A，D，C]的时候会进行全部对比，即按对应位置逐个比较，发现每个位置对应的元素都有所更新，则把旧集合全部移除，替换成新的集合，如上图，但是这样的操作在React中显然是复杂、低效、影响性能的操作，因为新集合中所有的元素都可以进行复用，无需删除重新创建，耗费性能和内存，只需要移动元素位置即可。


- **React对这一现象做出了一个高效的策略：允许开发者对同一层级的同组子节点添加唯一key值进行区分。意义就是代码上的一小步，性能上的一大步，甚至是翻天覆地的变化！**

4. 
![Element DIFF](https://img-blog.csdn.net/20180403132628782 )
同一个列表由旧变新有三种行为，插入、移动和删除，它的比较策略是对于每一个列表指定key，先将所有列表遍历一遍，确定要新增和删除的，再确定需要移动的。如图所示，第一步将D删掉，第二步增加E，再次执行时A和B只需要移动位置即可。


+ **在开发过程中，同层级的节点添加唯一key值可以极大提升性能，尽量减少将最后一个节点移动到列表首部的操作，当节点达到一定的数量以后或者操作过于频繁，在一定程度上会影响React的渲染性能。比如大量节点拖拽排序的问题。**



## Introduction

Diff two lists in time O(n). 
I
The algorithm finding the minimal amount of moves is [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) which is O(n*m). This algorithm is not the best but is enougth for front-end DOM list manipulation. 

This project is mostly influenced by [virtual-dom](https://github.com/Matt-Esch/virtual-dom/blob/master/vtree/diff.js) algorithm.

## Install

    $ npm install list-diff2 --save

## Usage

```javascript
var diff = require("list-diff2")
var oldList = [{id: "a"}, {id: "b"}, {id: "c"}, {id: "d"}, {id: "e"}]
var newList = [{id: "c"}, {id: "a"}, {id: "b"}, {id: "e"}, {id: "f"}]

var moves = diff(oldList, newList, "id")
// `moves` is a sequence of actions (remove or insert): 
// type 0 is removing, type 1 is inserting
// moves: [
//   {index: 3, type: 0},
//   {index: 0, type: 1, item: {id: "c"}}, 
//   {index: 3, type: 0}, 
//   {index: 4, type: 1, item: {id: "f"}}
//  ]

moves.forEach(function(move) {
  if (move.type === 0) {
    oldList.splice(move.index, 1) // type 0 is removing
  } else {
    oldList.splice(move.index, 0, move.item) // type 1 is inserting
  }
})

// now `oldList` is equal to `newList`
// [{id: "c"}, {id: "a"}, {id: "b"}, {id: "e"}, {id: "f"}]
console.log(oldList) 
```

