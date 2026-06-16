---
title: 凌晨三点的 console.log
subtitle: 关于一个被我写坏又写好的函数
date: 2026-06-12 02:30:00 +0800
author: 你的名字
tags: [JavaScript, debug]
---

## 起因

凌晨两点五十八分，窗外的灯全灭了，只剩屏幕。

我写了一个函数，处理用户上传的图片。逻辑很简单：判断大小、压缩、转码、存盘。
测了三遍都没问题。第四遍，加了一张 4K 的测试图——崩了。

错误信息：`Maximum call stack size exceeded`。

## 排查

我盯着那行代码看了十分钟。函数不复杂，十几行。看起来就是普普通通的递归处理——

```javascript
async function processImage(file) {
  if (file.size > MAX_SIZE) {
    const compressed = await compress(file);
    return processImage(compressed);  // ← 问题在这
  }
  return upload(file);
}
```

`Maximum call stack size exceeded` —— 栈溢出。看上去好像在递归上，但我只在文件超大时才递归一次，怎么可能溢出？

## 真相

原来不是递归的问题。

是 `compress` 函数里我自己写的那个 `await` 顺序错了——同一个 Promise 在多次调用之间没被正确处理，导致事件循环在某一次 `await` 处 hang 死，
等到下一次的 `processImage` 进来时，前一个还没出栈，就堆栈累积了。

说白了，是我把"异步"和"递归"两件事搅在一起，没想清楚它们的相互作用。

## 修复

```javascript
async function processImage(file) {
  let current = file;
  while (current.size > MAX_SIZE) {
    current = await compress(current);  // ← 改成 while 循环
  }
  return upload(current);
}
```

一行半的改动。

但我盯着这行半的改动，又看了十分钟。

## 后来

凌晨三点二十二分，console.log 打出来的那个数字终于对了。
我没欢呼，也没截图。关了电脑，去睡了。

第二天起来，那个函数又看了三遍，还是觉得不够好。但已经能用了，先这样。

---

> 写代码这事的浪漫之处就在于——
> 你以为你在写逻辑，其实你在跟自己对话。
> 每一个 bug 都是一面镜子，照出你哪一段思维没走通。