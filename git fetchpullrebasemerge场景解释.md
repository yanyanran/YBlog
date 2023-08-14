# git fetch/pull/rebase/merge场景解释

最近所开发的开源项目涉及到git相关操作，看源码时对git pull和git fetch的使用场景产生了疑惑。日常中开发往个人仓库里提交代码时用的都是add+commit+push常规操作，git fetch用的很少。

![](https://picx.zhimg.com/80/v2-af3bf6fee935820d481853e452ed2d55_1440w.webp?source=1940ef5c)



#### git pull两种组合：【fetch + merge】【fetch + rebase】

rebase干的事情就是变基，假如有这样一种情况：从master分支的最新结点为B时拉取了一个新分支feature，在feature分支上开发一段时间后想要拉取一下master分支上的最新状态，但此时master基于B已经提交了新结点头M：

![](https://img-blog.csdnimg.cn/36efc2704d174acab598c4b9addd3694.png?)

此时我们切换到feature分支上，执行rebase命令，相当于是想要把master分支合并到feature分支。此时rebase相当于将原本的基分支（B往前）提换掉了。

rebase的缺点是执行后就**不知道当前分支最早是从哪个分支拉出来的**了，因为基底变了。***【慎用rebase】***

