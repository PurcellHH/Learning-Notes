# Activity知识点
[TOC]
## 1. 概要
作为Android四大组件之一，Activity的使用频率是最高的。
Activity是个可视化的界面。可与用户进行交互。
## 2. 生命周期
下面是官方的生命周期图：
![](_v_images/20200406100914842_20076.png =547x)
### 2.1 正常的生命周期
正常情况下，Activity会进行一下方法回调：

* **onCreate():** 
表示Activity正在创建，此时可以做一些初始化操作，如setContentView界面资源、数据初始化。

* **onStart():** 
表示Activity正在启动，此时Activity可见，但不在前台，没有焦点，无法与用户交互。

* **onResume():** 
此时Activity获得焦点，出现在前台，可以与用户进行交互。

* **onPause():** 
表示Activity正在停止，此时可以做一些轻量级的数据存储、停止动画。

* **onStop():** 
表示Activity即将停止，此时可做稍微重量级回收工作，如取消网络连接、注销广播接收器等。

* **onReStart():** 
表示Activity重新启动，Activity由后台切换到前台，由不可见到可见。

* **onDestroy():** 
表示Activity即将销毁，此时做回收工作、资源释放。

Activity的生命周期：
> 1. 启动Activity-A：  A-onCreate > A-onStart > A-onResume
> 2. 打开Activity-B：  A-onPause > B-onCreate > B-onStart > B-onResume > A-onStop
> 3. 返回Activity-A：  B-onPause > A-onRestart > A-onStart > A-onResume > B-onStop > B-onDestory
> 4. 打开Dialog或透明主题的Activity-C:   A-onPause > C-onCreate > C-onStart > C-onResume
> 5. 关闭屏幕或者Home键：C-onPause > C-onStop > A-onStop
> 6. 点亮屏幕或回到前台：C-onRestart > C-onStart > A-onRestart > A-onStart > C-onResume
> 7. 关闭Dialog或Activity-C：C-onPause > A-onResume > C-onStop > C-onDestory
> 8. 销毁Activity-A：A-onPause > A-onStop > A-onDestory

### 2.2 异常的生命周期
#### 2.2.1 系统配置发生改变
#### 2.2.2 系统资源内存不足
## 3. 缓存机制
## 4. 启动模式
### 4.1 Standard
### 4.2 SingleTop
### 4.2 SingleTask
### 4.2 SingleInstance

## 5. 匹配规则
### 5.1 intent-filter
### 5.2 action
### 5.3 category
### 5.4 data
## 6.启动流程