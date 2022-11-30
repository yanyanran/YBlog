## 记一次git分支冲突的解决

最近准备开始写cs61b的lab了，发现三个月前开的远程仓库和本地仓库之间的更新有问题，打算重新pull一下结果报错：

![](https://github.com/yanyanran/pictures/blob/main/%E5%88%86%E6%94%AF%E5%86%B2%E7%AA%811.png?raw=true)

使用git status查看显示：

![](https://github.com/yanyanran/pictures/blob/main/%E5%88%86%E6%94%AF%E5%86%B2%E7%AA%812.png?raw=true)

git merge后add三件套push后显示当前提交落后于远程分支：

![](https://github.com/yanyanran/pictures/blob/main/%E5%88%86%E6%94%AF%E5%86%B2%E7%AA%813.png?raw=true)

按理来讲落后远程问题pull一下就好了，但pull之后的状态又回到了报错第一步（纯纯循环了属于是

然后我尝试在add和commit之后使用 git push **--force**：

![](https://github.com/yanyanran/pictures/blob/main/%E5%88%86%E6%94%AF%E5%86%B2%E7%AA%814.png?raw=true)

居然奇迹般push上去了，本地仓库的文件夹都更新到了远程中去。但是可以看到push完成结尾显示了什么鬼什么鬼权限不够。



> #### *插一条*：
>
> 当初绑仓库的时候搞成了http鉴权，导致每次push都要输密钥，麻烦的得很。把链接改成ssh就好了：
>
> > **cd .git   -->  vim config  // 修改里面的链接为ssh，保存退出即可** 

权限不够推测是报错文件的用户组搞错了，-l查看：

![](https://github.com/yanyanran/pictures/blob/main/%E5%88%86%E6%94%AF%E5%86%B2%E7%AA%815.png?raw=true)

果然，显示用户组是root，推测是之前clone仓库的时候加了sudo。在这样的情况下不加sudo push代码是无法进行的。

现在需要修改用户组，需要把这个仓库的用户组都修改为自己（yanran）。使用chown -R --recursive递归修改用户组

记得加sudo：

![](https://github.com/yanyanran/pictures/blob/main/%E5%88%86%E6%94%AF%E5%86%B2%E7%AA%816.png?raw=true)

over 权限不够的报错消失了，问题解决！

![](https://github.com/yanyanran/pictures/blob/main/%E5%88%86%E6%94%AF%E5%86%B2%E7%AA%817.png?raw=true)