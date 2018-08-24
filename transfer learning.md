---
title: 2018-8-24 迁移学习
梁子，20163933
2273067585@qq.com
---
>这份笔记学习自ＣＳ２３１Ｎ笔记中
# transfer learning 迁移学习
## 迁移学习的目的
仍然是为了抵制过拟合现象.使用迁移学习可以也是防止过拟合现象的一个途径.在小数据集上经常如此.
## 方法
对于小数据集,一般不会对参数部分再进行训练,而是只训练最后的分类部分,以来来获得做种比较好的拟合.
即使是对于较大的数据集,在训练时可以训练整个网络,但是会将学习略调的很低,因为这些模型之前已经在imageNet进行过训练,泛化能力已经足够,只不过是微调部分参数而已.
因而,迁移学习是是指:在相关的一个大数据集上已经经过了训练,之后在另一个任务上对其进行参数的微调,以实现效果.
如下图所示:.
![enter description here](https://www.github.com/liangzid/LittleBook/raw/master/小书匠/1535118061089.png)
关于采取何种策略,下图说明的很清晰了:
![enter description here](https://www.github.com/liangzid/LittleBook/raw/master/小书匠/1535118709855.png)
## 应用
迁移学习被应用于图像检测和图像理解上,如下图所述:
![enter description here](https://www.github.com/liangzid/LittleBook/raw/master/小书匠/1535118843607.png)
也就是说加载一个预先训练的CNN模型,之后再进行精细地调整.






