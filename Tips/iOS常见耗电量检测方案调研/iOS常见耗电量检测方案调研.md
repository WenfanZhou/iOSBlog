# iOS 常见耗电量检测方案调研


## 说明

如果我们想看下我们的 APP 或 SDK 是否耗电，需要给一些数据来展示，以下是对常见的电量测试方案做了一下调研。

影响 iOS 电量的因素，几个典型的耗电场景如下：

 1. 定位，尤其是调用GPS定位
 2. 网络传输，尤其是非Wifi环境
 3. cpu频率
 4. 内存调度频度
 5. 后台运行


## 系统接口

iOS 10 系统内置的 Setting 里可以查看各个 App 的电池消耗。

![enter image description here](https://www.imore.com/sites/imore.com/files/styles/larger/public/field/image/2015/10/ios-9-battery-usage-screens-01.jpg?itok=fGMOE3CR)

系统接口，能获取到整体的电池利用率，以及充电状态。代码演示如下：

 ```Objective-C
    //#import <UIKit/UIDevice.h>
    UIDevice *device = [UIDevice currentDevice];
    device.batteryMonitoringEnabled = YES;
    //UIDevice返回的batteryLevel的范围在0到1之间。
    NSUInteger batteryLevel = device.batteryLevel * 100;
    //获取充电状态
    UIDeviceBatteryState state = device.batteryState;
    if (state == UIDeviceBatteryStateCharging || state == UIDeviceBatteryStateFull) {
        //正在充电和电池已满
    }
 ```

这些均不符合我们的检测需求，不能检测固定某一时间段内的电池精准消耗。


## 测试平台

 **阿里云移动测试[MQC](http://mqc.yunos.com)**

[MQC](http://mqc.yunos.com) 调研，结论：没有iOS性能测试，无法提供耗电量指标。

解释 | 截图
-------------|-------------
安卓有性能测试项目| ![enter image description here](https://ws2.sinaimg.cn/large/006tNbRwly1fglofo7j2qj30p20ik0td.jpg) |
安卓的性能测试项目 |![enter image description here](https://ws1.sinaimg.cn/large/006tNbRwly1fglofo2v83j311g0cm74m.jpg) |
iOS没有性能测试，无法提供耗电量指标| ![enter image description here](https://ws1.sinaimg.cn/large/006tNbRwly1fglofnxmhvj31ba0ciq36.jpg)


百度移动云测试中心 MTC 同样没有 iOS 的性能测试。

其他测试平台类似。

## 常用的电量测试方法：

  1. 硬件测试
  2. 软件工具检测



## 软件工具检测

下面介绍通过软件 Instrument 来进行耗电检测。



## iOS电量测试方法

####  1.iOS 设置选项 ->开发者选项－>logging －>start recording

![enter image description here](https://ws4.sinaimg.cn/large/006tNbRwly1fgbkl24g4qj30eu08gjrk.jpg)

#### 2.进行需要测试电量的场景操作后进入开发者选项点击stop recording
#### 3.将iOS设备和Mac连接
#### 4.打开Instrument，选择Energy Diagnostics
#### 5.选择 File > Import Logged Data from Device

![enter image description here](https://ws1.sinaimg.cn/large/006tNbRwly1fgbkl20pt2j30ek08i3yv.jpg)


#### 6.保存的数据以时间轴输出到Instrument面板
![enter image description here](https://ws4.sinaimg.cn/large/006tNbRwly1fgbkl1w4rxj30fr0aajsv.jpg)

#### 其他

 - 测试过程中要断开 iOS设备和电脑、电源的连接
 - 电量使用level为0-20，1/20：表示运行该app，电池生命会有20个小时；20/20：表示运行该app，电池电量仅有1小时的生命
 - 数据不能导出计算，只能手动计算平均值

## Sysdiagnose 检测

官方的工具sysdiagnose。这是苹果日志系统的统称，苹果经常会询问是否要官方帮忙诊断和定位问题时，上传的就是sysdiagnose的各种日志。

Sysdiagnose很庞大，每天上几百M的日志，记录电池、第三方APP、各种系统功能和应用的所有运行情况。

Sysdiagnose怎么使用呢？简单得说，就是需要一个开发者账号，然后到苹果开发者网下载对应的证书。不需要越狱，没有系统限制，这个特别关键。

电量日志是sysdiagnose系统中最庞大的一块。电量日志每天有几十到一百M，他是一个庞大的数据库，里面有267张表，记录了电池电量的各维度信息。

![enter image description here](https://blog-10039692.file.myqcloud.com/1508982751904_5537_1508982985887.png)

- 测试方法

进行电量测试，只要装上对应的证书，便可开始执行测试，只要记下哪个时间段对应的是哪个场景，然后测试完后，取下系统的数据库，便可以对当次的电量做较全面的评价，例如，在某场景下，20分钟运行时间，显示耗电100mWh，CPU耗电20mWh，运行温度是32度，平均电流是110mA，这样的数据可以具体的体现出APP的耗电情况。

## 硬件检测

 通过硬件 [PowerMonitor]( https://www.msoon.com/LabEquipment/PowerMonitor/ ) 可以精准地获得应用的电量消耗。

 步骤如下：

  1. 拆开iOS设备的外壳，找到电池后面的电源针脚。
  2. 连接电源监控器的设备针脚
  3. 运行应用
  4. 测量电量消耗

 下图展示了与iPhone的电池针脚连接的电源监控器工具。

 ![enter image description here](https://bottleofcode.com/wp-content/uploads/2015/06/9.png)

 可以参考：[**Using Monsoon Power Monitor with iPhone 5s**]( https://www.bottleofcode.com/2015/07/12/using-monsoon-power-monitor-with-iphone-5s/)。

 - 可以精准地获得应用的电量消耗。
 - 设备价格 $771.00 USD
 - 需要拆解手机

## 结论
手机通过代码方式无法直接获取APP的耗电情况，PowerMonitor应该有专业的检测机构可以测试。
