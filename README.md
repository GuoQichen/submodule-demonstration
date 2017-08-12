## 目的
1. 分开管理
2. 如果是添加公共的库到当前项目中，比起直接复制代码，submodule能追踪上游的更新

## 添加submodule
```
git submodule add <repository> [<path>]
```
添加后会多一个`.gitmodules`文件，文件的内容只是保存submodule的引用信息，包括路径和repo地址

```
[submodule "simple-todos"]
	path = simple-todos
	url = git@github.com:GuoQichen/simple-todos.git
```
添加完后需要在主项目`git commit `提交新增子项目文件夹以及`.gitmodules`文件的修改

在子项目加入到项目的时候，其实做了这样三件事：
1. 记录引用的子项目仓库，在`.git/config`中
2. 记录当前项目中子项目的目录位置，在生成的`.gitmodules`中
3. 记录子项目的**commit id**

所以在当前项目push到remote repository的时候，只是更新了引用的`commit id`，那么在其他人clone项目的时候，就可以获取子项目的`commit id`，然后在`git submodule update`的时候获取子项目`commit id`所表示的commit

## clone带有submodule的项目

查看子模块的commit id，如果子模块没有被checkout，前面会有`-`，那么就需要`git submodule init`， `git submodule update`

```
git submodule
// result
b6bf4d6a6cbaaff39e4e4d4f3108d16267354844 simple-todos (heads/master)
// or
-b6bf4d6a6cbaaff39e4e4d4f3108d16267354844 simple-todos (heads/master)
```

clone带有submodule的项目的两种方法:
1. 第一种
```
git clone <repository>
git submodule init
git submodule update
```
2. 第二种
```
git clone <repository> --recursive
```
其实相当于
```
git submodule update --init --recursive
```

注意，在clone之后，`cd submodule/`，然后`git status`，你会看到这样的状态
```
HEAD detached at b6bf4d6
nothing to commit, working tree clean
```
也就是说，现在是`detached HEAD`，使用`git branch`就可以查看到
```
> * (HEAD detached at b6bf4d6)
> master
```
所以需要`git checkout master`到master分支，然后在master分支上修改

为什么呢？因为父项目不记录子模块的修改，只记录commit id，所以clone的时候只获取到对应的commit，而不在任何分支上，但是master分支的commit id和HEAD保持一致，所以只要`git checkout master`，而不需要新建分支

## 修改submodule
注意：**只有子项目内容更新，就需要更新父项目引用的子项目的commit id**

其实子项目和父项目只是独立的git项目，所以其他操作和一般git项目一样，在子项目修改，需要
```
cd submodule/
git add .
git commit - m 'xxx'
git push
```
其实普通的git项目一样操作，然后需要在父项目更新引用的子项目commit id，有~~两~~一种方法：
1. ~~`git submodule update`~~
~~但是现在的子模块指向的不是master分支，而是commit id表示的commit，所以还需要~~
<del>
<pre>
<code>
cd submodule/
git checkout master
// todo
</code>
</pre>
</del>
为什么这种方法不行呢，其实是个坑！就是你`git submodule update`之后，你的子项目会恢复到你父项目引用的那个commit，也就是不是最新的commit，此时`git status`是干净的，就像第一次`git clone`项目然后`git submodule init`，`git submodule update`一样，子项目指向的是父项目引用的那个commit，所以此时把子项目`git checkout master`之后，再切回到父项目，使用`git status`，你会发现提示子项目有新的commit，所以还是需要父项目更新引用的子项目的commit id

1. 在父项目更新引用的子项目commit id
```
git commit -m 'update submodule commit id'
git push
```

## 更新submodule
注意：**如果更新submodule的时候有新的commit id产生，需要在父项目产生一个提交，用来更新对子项目commit id的引用**
两种方法:
1. `git submodule foreach git pull`
直接在父项目运行，这样能一次性把所有子项目都更新
2. 到子项目中去更新
```
cd submodule/
git pull
```

## 总结
子项目一旦产生变动，有新的commit id，父项目必需产生一个提交，更新对子项目commit id的引用

## 参考
1. [Git Submodule使用完整教程](http://www.kafeitu.me/git/2012/03/27/git-submodule.html)
2. [使用Git Submodule管理子模块](https://segmentfault.com/a/1190000003076028)
3. [Git submodule的坑](http://blog.devtang.com/2013/05/08/git-submodule-issues/)
