## 1.网站是怎么搭建的
http协议

网络的三种架构：客户机/服务器结构（C/S）例如：QQ、微信 浏览器/服务器（B/S）例如：百度   P2P结构  例如：cs&#x20;

![](https://note.youdao.com/yws/res/3/WEBRESOURCE2a931c8179c3bf8bcf8f7750f2c9fa33)C/S	需要安装客户端软件，客户端与服务器直接通信

优点：交互体验好（可离线部分功能）、速度快

缺点：客户端需要单独开发维护，多端设备适配成本高、更新需用户手动升级

B/S	无需安装客户端，通过浏览器访问

优点：跨设备通用、更新只用服务器端升级

缺点：依赖网络、体验搜浏览器性能限制

P2P	无中心服务器，设备间直接通信

优点：去中心化、抗单点故障、资源共享效率高

缺点：安全性差、节点直接暴露、管理困难

![](https://note.youdao.com/yws/res/c/WEBRESOURCEdd5add0836866e31befdeada0e1eea8c)![](https://note.youdao.com/yws/res/3/WEBRESOURCE4c545d5175f551d81339426b154bae33)网站必需五大件：域名 服务器 前端 后端 中间件

![](https://note.youdao.com/yws/res/d/WEBRESOURCEc5601a12cb9a3578033e0bf274b4605d)![](https://note.youdao.com/yws/res/2/WEBRESOURCE020c24e392353a4df8fa1a5ef5d29032)

大型服务器应能7*24h运行

![](https://note.youdao.com/yws/res/3/WEBRESOURCE1b004284c4409a7f51cab8447f847893)

常用中间件：APACHE（阿帕奇)、Nginx、Tomcat、IIS

![](https://note.youdao.com/yws/res/6/WEBRESOURCE87b00897a1a95b79d1e189b64f14aba6)![](https://note.youdao.com/yws/res/6/WEBRESOURCE2f81e9639d7e69f97682f0e6efbd4a66)![](https://note.youdao.com/yws/res/1/WEBRESOURCE62a8e53cc5f2f1805d1210fe436133b1)

域名  ：[www.baidu.com](http://www.baidu.com)   对应IP：14.119.104.154

	正常访问需要网站的IP，但我们只需要输入域名，

访问时输入域名，通过DNS服务器将域名与对应的IP相转换，即DNS解析，DNS类似于翻译

域名好记且具有商业化价值

![](https://note.youdao.com/yws/res/5/WEBRESOURCEacd7c379a84c991d4bd4443fc077c755)

## 2.操作系统
window、Linux（命令提示符、但可以图形可视化）

区别：内核不同

![](https://note.youdao.com/yws/res/a/WEBRESOURCEd03e0a218c2f3e1f87d721e8d41e7a1a)![](https://note.youdao.com/yws/res/5/WEBRESOURCEc2310f2a86d98d196e58d1d3c9eda8a5)![](https://note.youdao.com/yws/res/3/WEBRESOURCE04cb5f829e617ebf2e90607f18508083)

## 3.安装VMware
VMware 虚拟机 可在一台电脑上运行多个电脑

好用的功能：快照（类似游戏存档——可读取回档）快照要内存

vmtools——修改屏幕分辨率——物理电脑和虚拟电脑文件传输

可用远程桌面代替vmtools，不要钻牛角尖，要先在设置中开启

win+r 输入mstsc  输入ip 登入

虚拟机网络  ：

桥接模式 将当前用的网络分一个给虚拟机 ip前3位地址相同 在同一个局域网 可相互通信访问

nat模式  IP前3位地址不同 转换了IP地址 不在同一个局域网 不可相互通信访问 安全性高

两台虚拟机的通话：虚拟机网络模式相同

1.全部桥接

2.全nat

可用 命令提示符中 用ping ip来确定

