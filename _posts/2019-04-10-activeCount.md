---
layout:       post
title:        "Thread.activeCount()的值"
subtitle:     "Thread.activeCount()的值在不同IDE中是不一样的"
date:         2019-04-10
author:       "Rong"
header-mask:  0.3
catalog:      true
tags:
    - Java
    - Thread
    - 多线程
---

这几天一直在看多线程相关的东西，今天写了个多线程读取文件并行计算然后汇总数据的demo，发现在IDEA中`Thread.activeCount()`的值不是理想的1。

直接上码：

```java
public static void main(String[] args) {
		List<String> contents = readFile();
		int lineCount = contents.size();
		Demo d = new Demo(lineCount);
    // 多线程小汇总数据
		for (int i = 0; i < lineCount; i++) {
			final int j = i;
			new Thread(new Runnable() {
				@Override
				public void run() {
					d.calc(contents.get(j), j);
				}
			}).start();
		}
    // 等待小汇总任务完成
		while (Thread.activeCount() > 1) {
		}
    // 汇总总数据
		d.sum();
	}
```

在debug模式下是理想的，执行完所以的小汇总数据任务后才执行sum(),可在run的时候死活不执行。我`jconsole`和`jvisualVM`查看了线程也没看出个所以然。[stackoverflow搜索](https://stackoverflow.com/questions/30561424/thread-count-is-different-when-debugging-or-running-on-intellij)，原来是idea的锅，再看线程确实有`Monitor Ctrl-Break`

最后，可以使用`CountDownLatch`来实现多线程通信。

撒花*★,°*:.☆(￣▽￣)/$:*.°★* 。