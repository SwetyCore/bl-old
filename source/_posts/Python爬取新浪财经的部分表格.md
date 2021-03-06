---
layout: post
title: Python爬取新浪财经的部分表格
categories: [爬虫]
date: 2021-07-01 00:00:00
keywords: python, requests
---

# 前言

需求：根据提供的股票代码将网站 [财务摘要_新浪网](http://money.finance.sina.com.cn/corp/go.php/vFD_FinanceSummary/stockid/600000/displaytype/4.pt.phtml) 的资产负债表，利润表，现金流量表三个表格保村到本地。

# 过程解析

得到需求后当然是打开浏览器，然后根据传统艺能打开 DevTools 就是一顿分析，很快嗷，就得到了爬取的思路。

1.  获取相关页面：对 `http://vip.stock.finance.sina.com.cn/q/go.php/vInvestConsult/kind/scbhz/index.phtml?symbol={股票代码} `    发送get请求，得到搜索结果。用xpath`//a[@target="_blank"] `来匹配到财务摘要的页面。
2. 匹配相关信息：在财务摘要页面用 `//*[@id="toolbar"]/div[1]/h1/a` 来获取公司名称，正则匹配 `stockid/(.*?)/ctrl`  来获取stockid。
3. 获取下载链接并下载：其中，资产负债表，利润表，现金流表的下载链接分别是 `http://money.finance.sina.com.cn/corp/go.php/vDOWN_BalanceSheet/displaytype/4/stockid/{stockId}/ctrl/all.phtml`   `http://money.finance.sina.com.cn/corp/go.php/vDOWN_ProfitStatement/displaytype/4/stockid/{stockId}/ctrl/all.phtml`   `http://money.finance.sina.com.cn/corp/go.php/vDOWN_CashFlow/displaytype/4/stockid/{stockId}/ctrl/all.phtml` 

# 遇到的问题&吐槽

获取页面和匹配信息都没什么难度，令人迷惑的是当我在浏览器里试图下载 excel 文件的时候，突然弹出了登录框提示登陆才能下载，当时猜测完了可能又要搞模拟登陆了（心里不由得想起了被各种加密支配的恐惧）。但当我真正登陆上去点击下载后，才发现不就一固定链接嘛还搞得那么神秘。



项目地址：
