---
layout: post
title: Python 模拟登陆
categories: [爬虫]
date: 2021-03-25 00:00:00
keywords: python, requests
---

# 前言：
这个东西其实在寒假期间就写好了，但是每天都需要手动的健康打卡，但是自己又是非常的懒，一不小心睡过头或者玩过头了就把这件事忘的一干二净，当时虽然已经学了一点爬虫的方法，但是也只会爬点文字，图片一类的东西，对于需要登陆才能获取到信息的网站没有一点办法。自动打卡首先需要登陆综合门户，那就硬着头皮面向百度编程吧[无奈]


## 过程解析:
1. 初次请求登录门户(get):
>https://pass.ujs.edu.cn/cas/login
2. 请求验证码(get): 
>https://pass.ujs.edu.cn/cas/captcha.html
3. 再次用请求登录门户 (post) ,并提交登录表单.

三次请求如果都成功并且无错误,应该返回成功登陆后的界面.

### post表单内容解析：
```
username: 1234567890
password: gfgwDALwERl0MuTBfg+U/m81JBDbksfasfTj8msH9HqqKztL+3lIhePVsFD5GXoDi7DYyQs7YoeN1Yz0RlMhegECvXfiIsQ=
captchaResponse: aphc
lt: LT-958050-a0kzJW9kweaskdgfHJdaY1Ug7oHe2eQ1611227557350-OC02-cas
dllt: userNamePasswordLogin
execution: e2s1
_eventId: submit
rmShown: 1
```
+ username: 学号
+ password: 加密后的密码
+ captchaResponse: 验证码
+ lt: 一串奇怪的字符,在第一次请求的 response 里面,可以用正则表达式 `<input name="lt" type="hidden" value="(.*?)"/>` 匹配到.
+ dllt: 固定值
+ execution: 奇怪的字符,使用正则表达式 `<input name="execution" type="hidden" value="(.*?)"/>` 匹配
+ _eventId: 固定值
+ rmShown: 固定值

### 密码加密解析:
只是将网页的js代码稍作整理提取,有兴趣的大佬可以研究一下具体的加密过程(其实是我不会).
其中的加密函数 `_etd2()` 需要
两个参数:

+ 密码
+ pwdDefaultEncryptSalt

其中pwdDefaultEncryptSalt可以用正则表达式 `var pwdDefaultEncryptSalt = "(.*?)";` 匹配.

我已经将加密的代码整理为两个js文件,使用时候只需要运行jiami.js里面的 `_etd2()` 函数并且传入相应的参数,获取函数返回值即可.

## 注意事项
+ 三次请求应该是同一个 session 完成

## 请求示例:
~~详见login.py~~

重写了 login.py ,建议使用 loginRe.py
+ 调用登陆模块示例: 



 ```python
import loginRe
import getpass

if __name__ == '__main__':
    username = input('学号: ')
    pwd = getpass.getpass("密码: ")
    lg = loginRe.Login('6e9b090sdad5c1234d0bbb88de18097eb')
    lg.Login(username=username, password=pwd)

 ```
#### tip:
阿里云 ocr 识别请自行申请接口 key. [戳我传送](https://market.aliyun.com/products/57124001/cmapi020020.html)

模拟登陆的用途:应该没什么用吧 (bushi)

仅供学习交流,请不要用于其他方面!!!!

希望不至于被叫去喝茶 (逃)



# 总结：
虽然这个项目的难度并不高，但是我对于需要登陆的爬虫的第一次尝试。
我竟然笨到研究了三天才整完。
项目详见 [Jiangda-Portal-Automatic-Login](https://github.com/SwetyCore/Jiangda-Portal-Automatic-Login)。

