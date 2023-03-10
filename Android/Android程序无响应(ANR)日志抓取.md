## 1. 什么是ANR

**ANR，英文全称Application Not Responding，中文释义为应用程序无响应**。当应用程序有一段时间**响应不够灵敏**时，系统会向用户展示一个对话框，用户可以选择”等待“让程序继续运行，或选择“关闭应用”来强制关闭应用。

一个流畅的合理的应用程序中不能出现ANR，ANR会导致用户体验变差。默认情况下，在Android中Activity的最长执行时间是5秒，BroadcastReceiver的最长执行时间则是10秒。

![微信截图_20200329144721](Android程序无响应(ANR)日志抓取.assets/微信截图_20200329144721.png)

## 2. 为什么会出现ANR

在Andorid系统中，通过Activity Manager和Window Manager服务来监控应用程序的响应情况，如果应用程序响应时间超出了限定时间，为了避免对用户体验造成困扰，系统会弹出ANR对话框来让用户选择是否继续等待。

**出现应用程序响应时间超出限定时间主要有以下几种情况**

* **主线程阻塞**

  在主线程中执行了大量耗时的操作，比如网络请求（现在会抛出网络请求不能放在主线程的异常），从数据库中读取大量数据，线程长时间休眠。

* **CPU满负荷工作时进行I/0操作**

  当CPU满负荷工作时，如果APP仍旧进行一些I/O操作，会导致ANR

* **内存不足**

  系统分配给每个应用程序的内存空间是一定的，如果APP中发生内存泄露等，会导致ANR

## 3. ANR日志抓取

当应用程序发生ANR后，系统会在**data/anr/**目录下生成一个**traces.txt**文件，文件内记录了ANR产生时的一些系统信息，我们可以通过使用adb命令来将文件导出至本地。

在导出之前，我们可以先查看下设备ANR日志是否存在

![1585466042439](Android程序无响应(ANR)日志抓取.assets/1585466042439.png)

**使用如下命令来将ANR日志导出**

~~~
adb pull data/anr/traces.txt 导出后的路径
~~~

例如：

![1585466215165](Android程序无响应(ANR)日志抓取.assets/1585466215165.png)

然后我们打开traces.txt进行分析

![1585466589068](Android程序无响应(ANR)日志抓取.assets/1585466589068.png)



---

GitHub：[https://github.com/dkbnull/AndroidDemo](https://github.com/dkbnull/AndroidDemo)

Gitee：[https://gitee.com/dkbnull/AndroidDemo](https://gitee.com/dkbnull/AndroidDemo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/105179536](https://blog.csdn.net/dkbnull/article/details/105179536)

微信：[https://mp.weixin.qq.com/s/CCh7HghMKkZDGSWiBeCbMA](https://mp.weixin.qq.com/s/CCh7HghMKkZDGSWiBeCbMA)

知乎：[https://zhuanlan.zhihu.com/p/613031195](https://zhuanlan.zhihu.com/p/613031195)

---

