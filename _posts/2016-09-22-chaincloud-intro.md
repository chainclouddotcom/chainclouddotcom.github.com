---
layout: post
title: "ChainCloud 开发介绍"
date: 2016-09-22
desc: "介绍如何基于 ChainCloud 来开发您的区块链平台"
keywords: "bitcoin，chain"
categories: [business]
tags: [bitcoin]
---

# 1. 开发

## 1.1 发币

![intro-01]({{ site.img_path }}/picture/intro-01.jpg)

### 1.1.1 如何给一个地址发币？

发币的过程是通过 V设备 来实现的，你需要提供两个 API 给 V设备 来完成发币流程。

V设备 会轮询你的‘待发送的交易’的 API，查看是否有待发送的交易；

查到有待发送交易后，V设备 就会与 ChainCloud 进行交互，并签名、广播这笔交易，然后把tx_hash反馈给你的‘更新交易状态’ API。

* ‘待发送的交易’ API 的编写方法： http://docs.chaincloud.com/en/latest/chaincloud-v/v-device-sample.html#get-a-tx-to-be-sended

* ‘更新交易状态’ API 的编写方法： http://docs.chaincloud.com/en/latest/chaincloud-v/v-device-sample.html#update-a-tx-status

你可以这样做：

创建一个待发交易表sending_tx，有新的发币请求则向该表中插入一条数据，‘待发送的交易’、‘更新交易状态’api对这个表进行查询、更新操作。

### 1.1.2 交易加密（提高安全性）

如果交易要用 AES 加密，请按照我们的加密逻辑写，我们有 java、python、js 版本的实现；如果你使用的不是这几种语言，那你可能需要参考这几个版本的逻辑来自行实现。

如果能跑通以下例子，则就能保证 V设备 的 AES 解密逻辑与你服务器加密逻辑一致。

* 源字符串：{“outs”:”1NbbvxBYxGGCBhaM8mow1HFWA7dB5yukmY,10000;1MCs9SZwLg9JvLo6pzvVBWtmSV1dakwyM1,10000”,”dynamic”:0}

* 加密后：B23A7DDF7AF953D4344ED8B190E9424C2A8D85CAA00B4358D58C5341E36A194097EA21F7D457E770E2C9512834D206DE9EB766ED154F7ECEABB61056E97D200F174EAD69CF62E5A7EDDF823AB8293EAEDFA47AA1FF66D1FDF2D4D2B6FFC0DF5E1D62DB84C7A3C6E432CDE0CDEABB9C79/166EAB021AFB6244191EEE75340F8109/D267F338A21B487D

* 密码：20160721

## 1.2 收币

### 1.2.1 如何知道你的冷地址中收到币？

你需要轮询这个 API 来查看你的冷地址是否收到币： http://docs.chaincloud.com/en/latest/api/basic-api.html#tx-list

你可以这样做：

每隔10秒钟，查询这个 API 查看是否有新的交易（参数：最后一次处理的 tx_hash，第一次为 null），如果有新的交易则根据 input，output 进行处理。

所以你要维护上次处理交易的 tx_hash;

### 1.2.2 地址

分类：冷收币地址（收币用）；热发币地址（往 ChainCloud 账户充钱用）。

ChainCloud 提供的地址是按批次返回的，一个批次1000个。如何使用这些地址则需要你自己维护。

获取批次地址 API（冷收、热发地址用各自token即可）是： http://docs.chaincloud.com/en/latest/api/basic-api.html#address-batch

为确保 ChainCloud 返回的地址正确性，你应该对 API 返回的地址进行校验。地址校验也是按批次校验的，1000个地址/批次。

### 1.2.3 如何进行地址校验？

地址的过程是通过 V设备 来实现的，你需要提供两个 API 给 V设备 来完成地址校验。

V设备 会自动轮询你的‘待校验批次地址’的 API，查看是否有待校验的地址；

查到有待校验地址 V设备 就会直接与 ChainCloud HSM-Cold 冷设备进行交互，获取对应批次的地址校验码，然后把校验结果反馈给你的‘更新校验结果’的 API

* ‘待校验批次地址’ API 的写法： http://docs.chaincloud.com/en/latest/chaincloud-v/v-device-sample.html#get-a-batch-addresses-to-be-checked

* ‘更新校验结果’ API 的写法： http://docs.chaincloud.com/en/latest/chaincloud-v/v-device-sample.html#update-a-batch-addresses-status

你可以这样做：

* 创建一个 batch 表，一个 address 表，batch 跟 address 一对多关联。batch 表中有是否需要校验和校验结果的字段。

* 获取批次地址，插入 batch、address 表。‘待校验批次地址’、‘更新校验结果’ API 对 batch、address 表进行查询、更新操作。

# 2. V设备 的使用

## 2.1 第一步 安装并启动应用 (会提示 token 过期，可忽略)

![intro-02]({{ site.img_path }}/picture/intro-02.png)

## 2.2 第二步 配置应用

### 2.2.1 设置 token（你的服务器 token、ChainCloud 热发款账户的 token、ChainCloud 冷收款账户的 token）

![intro-03]({{ site.img_path }}/picture/intro-03.png)

* ‘vtest 的 token’指的是提供发币、地址校验相关的四个 API 的服务器 token（可选，建议加 token，token 的方式跟调用 ChainCloud一样，在 http header 中加 token）

* ‘热发款账户的 token’指的是 ChainCloud 热发款 token

* ‘冷收款的 token’指的是 ChainCloud 冷冷收款 token

### 2.2.2 设置 channel（设置 ChainCloud手机号、你的管理员手机号）

![intro-04]({{ site.img_path }}/picture/intro-04.png)

* ’ChainCloud 手机号’指 ChainCloud 会给你一个手机号，通过该手机号来保证目前数据的安全、地址校验等；V设备 当前只支持设置1个 ChainCloud 手机号，这里只能填‘ChainCloud 手机号1’这个输入框；

* ‘管理员手机号’指你需要一个手机号来接受一些安全通知、发送一些指令给 V设备 等；

* ‘建立通道’指输入 ChainCloud手机号后，ChainCloud 与 V设备 要通过短信建立一个安全通道。而且 V设备 每天会自动更新安全通道。但是你必须要手动打开‘读短信’、‘写短信’、‘修改短信’的权限。设置－>应用->权限 里，具体设置方式不同种类手机略有不同。

### 2.2.3 设置传输密码（待发交易 AES 加密密码，可选）

![intro-05]({{ site.img_path }}/picture/intro-05.png)

发币相关 API 数据支持 AES 加密，如果需要启用 AES 加密，需要在这里设置密码。

### 2.2.4 vweb 域名（热发币、地址校验相关4个 API 的服务器地址）

![intro-06]({{ site.img_path }}/picture/intro-06.png)

热发币、地址校验相关4个 API 的服务器地址（端口如果不是80，要带上端口号）

* ‘开始LOOP地址校验’指启动地址校验服务，每5分钟一次请求。启动前必须把地址相关的api按照文档完成

* ‘停止LOOP地址校验’指停止地址校验服务。

## 2.3 第三步 启动热发币服务

![intro-07]({{ site.img_path }}/picture/intro-07.png)

* ‘最新vc通道ID’显示当前可用通道的ID

* ‘绑定服务’只有绑定服务才能进行其他操作

* ‘取消绑定’当操作完后，就可以取消绑定，这样的效率会更高

* ‘开始loop’指循环获取待发交易，每隔3秒一次

* ‘结束loop’指停止循环获取交易

* ‘测试一条数据’指运行一次 获取交易->chaincloud签名广播->反馈发币结果

* 'vc ping'测试通道是否建立、是否通畅。可以用此检测通道是否通畅。建立完通道后，可以以此来测试是否建立成功

* 下面的黑框为控制台，如果‘绑定服务’，它会实时显示当前运行状态
