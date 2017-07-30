---
title: Understanding Machine Learning With Python
date: 2017-07-18 22:12:12
tags: MachineLearning, Python
---

# 环境准备
学习环境我选择了Jupyter Notebook， 原因是它既可以使用Markdown去写笔记，同时也可以直接运行Python代码并显示代码的执行结果，非常的方便。

下载安装Anaconda[https://www.continuum.io/downloads](https://www.continuum.io/downloads), 这里我们选择了Python 3.x版本。Anaconda整合了机器学习的Library，一键安装快速无痛。安装完成后，提示是否将Anaconda添加到系统环境变量，软件默认不建议添加，原因是会将Anaconda添加到已有的环境变量之前，会导致一些冲突。 官方推荐从开始菜单中选择*Jupyter Notebook*应用来启动。这里我选择了手工将Anaconda3\Scripts目录添加到用户的Path里面，这样我就可以命令行中切到某个指定存放笔记的目录下面，再启动notebook程序，就可以将已有的笔记直接加载好。

在windows command line 中启动Jupyter Notebook：
> D:\Notebook\jupyter notebook

在浏览器中打开地址 http://localhost:8888 创建一个Python的note
<!-- more -->
![](http://res.cloudinary.com/delong/image/upload/v1500390036/jupyter_notebook_j2scrt.png)

# 机器学习的分类
## 有监督的机器学习
有监督的机器学习是指已知一部分数据，通过有监督的机器学习算法训练得出一个模型，再将模型应予于未知数据。
有监督指适用于分类和回归问题。
## 无监督的机器学习
无监督机器学习适用于聚类，从无序的数据里分析出那些数据属于一类。比如几个人的声音混合在一起，通过无监督机器学习的方法将他们分离。
feature_col_names = ['num_preg']