---
layout: post
title: Python实现博客文章的上传
categories: [爬虫]
date: 2021-03-26 00:00:00
keywords: python, requests
---


# 前言
在我刚开始创建这个博客的时候，每次发文章都需要将整个项目pull下来再push上去，十分不方便，于是就想做一个能够不用git就实现的快速文章上传工具，去网络上一搜，果然有个`github3`的模块可以实现。
# 过程
1. 导入模块：  

	`import github3`  
2. 实现登陆：

	```python
	def connect_to_github():
		try:
			gh=login(token=input("Input your token:"))
			repo=gh.repository("SwetyCore","swetycore.github.io")
			branch=repo.branch("main")
			return gh, repo, branch
		except Exception as msg:
			print(f'Token error!!!\n{msg}\nPlease retry.')
			connect_to_github()
	```
3. 上传文章：

	```python
	def store_data(data,filename):
		gh, repo, branch = connect_to_github()
		remote_path = f"_posts/{filename}"
		repo.create_file(remote_path, "Commite message", data)
		print("upload success")
		return
	```

# 吐槽
+ 为什么网上的相关教程这么少哇
+ 网上的教程中的`tree = branch.commit.commit.tree.recurse()`已经不再适用。
	所以判断文件是否存在应该改为
	```python
	def is_file_exists(filepath):
		gh, repo, branch = connect_to_github()
		tree = branch.commit.commit.tree.to_tree().recurse()
		for filename in tree.tree:
			if filepath in filename.path:
				return True
			else:
				return False
	```
+ 上传文件不能重名！！！！(以后再想办法解决吧【恼】)

