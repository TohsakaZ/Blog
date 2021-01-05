---
title: Flask -- Register功能实现
p: _posts/Python/Flask_register
date: 2021-01-06 00:28:19
tags:
---


### 基于Flask的用户注册功能实现
总结一下实现用户注册功能
<!--more-->
基本功能逻辑：提供注册用户页面，用户填写相应信息提交表单后，在后台创建相应用户，同时通过邮件发送确认连接，在用户确认之前应该都无法访问其他页面，都应跳转到提醒需要点击确认连接的页面。

#### 关键点
* 确认连接的实现
  * 正常思路是不同用户应该映射有不同的确认连接，当用户和链接匹配的时候，确认过程完成。因此，用户点击确认连接应该是处于登录的状态
  * 如何实现映射？一个显然思路是根据用户id，但缺点在于太容易被猜到规律，因此使用itsdangerous库生成复杂无规律字符串

* 在用户注册成功但未确认之前应该无法访问其他页面
  * 一个显然的思路，在视图函数前加装饰器，但是整个app的视图函数非常多，一一加不现实
  * 使用flask提供的before_app_request函数，其能够在用户发送请求之前，先调用其装饰的函数，函数说明如下：
  ```python
    def before_app_request(self, f):
      """Like :meth:`Flask.before_request`.  Such a function is executed
      before each request, even if outside of a blueprint.
      """
      self.record_once(lambda s: s.app.before_request_funcs
          .setdefault(None, []).append(f))
      return f
  ```
  可以看到，这里是提供给Blueprint类的函数，但是其作用范围超出了当前的Blueprint对应的视图函数，即整个app


接着，依然是基于典型的MVC思路构建功能。
* M（model）主要对应相应用户功能
* V (view) 对应相应的注册页面
* C(control) 对应相应的视图函数

#### View界面设计
主要涉及相应的表单即可，直接套用wtf-form，需要注意在login页面提供进入Register的入口

#### Model更改
这里Model主要是相应的用户模型，由于注册需要认证机制，所以需要新增confirmed属性。同时，提供生成映射确认连接的函数，以及根据确认连接匹配的函数

#### Control
* 增加处理注册页面视图函数
* 增加处理确认连接的视图函数
* 增加注册成功但未确认的函数（即处理未确认页面的视图函数）


