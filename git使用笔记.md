# 切换账户

## 不用ssh切换账号

- 查看本地的用户及邮箱

    `git config user.name`

    `git config user.email`

- 修改账户

    `git config --global user.name "username"`

    `git config --global user.email "email"`


# 新建、切换、合并分支

- 新建分支

    `git branch xxx`

- 切换分支

    `git checkout xxx`

上面两条命令可以用一条命令实现**新建并切换分支**

 `git checkout -b xxx`

 - 合并分支

    首先切换到需要合并到的分支，执行

    `git merge xxx` xxx是需要合并过来的分支名

# 发生冲突

## 内容冲突

出现冲突时会出现**CONFLICT**字样，而且此时分支并不是在某一分支而是在**master|MERGING**；

![git 内容冲突](http://orzoelfvh.bkt.clouddn.com/git%20%E5%90%88%E5%B9%B6%E5%86%B2%E7%AA%81.png?attname=&e=1498205923&token=cs2nCfx72Y7hW0_NpFYzb3Jab90IJWraRtphMd-q:p3zSW1cLxlaHROLLVYT_o1v_ym4)

最简单的解决办法是查看冲突的文件，例如上图为*git test.txt*,打开后可以发现：

![git 内容冲突内容](http://orzoelfvh.bkt.clouddn.com/git%E5%86%85%E5%AE%B9%E5%86%B2%E7%AA%81%E5%86%85%E5%AE%B9.png?attname=&e=1498206071&token=cs2nCfx72Y7hW0_NpFYzb3Jab90IJWraRtphMd-q:RLFdtKpKUIQVFYWOMg4mos0kllA)

`<<<<<<<`和`>>>>>>>` 中间就是发生冲突的地方，此时直接编辑冲突文件，然后把`<<<<<<<`和`>>>>>>>`以及中间的等号删除，然后再执行命令`git add .`以及`git commit -m '注释'`就解决了冲突。

# 发生Please enter a commit message to explain why this merge is necessary.

这句话的意思就是需要提交消息解释为什么合并是必要的。

此时会弹出VIM界面如图：

![合并冲出出现vim界面](http://orzoelfvh.bkt.clouddn.com/%E5%90%88%E5%B9%B6%E5%86%B2%E7%AA%81%E5%87%BA%E7%8E%B0vim.jpg?attname=&e=1498206503&token=cs2nCfx72Y7hW0_NpFYzb3Jab90IJWraRtphMd-q:SoFxxTDV6yk_odh48Ke-EsqRjEY)

此时可以做如下操作

- 按键盘字母 i 进入insert模式

- 修改最上面那行黄色合并信息,可以不修改

- 按键盘左上角"Esc"

- 输入":wq",注意是冒号+wq,按回车键即可

退出这个界面的话按`ctrl + z`；


# 放弃合并

如果合并以后未进行提交，则这个时候分支是处于中间状态，直接执行命令：`git merge --abort` 就可以撤销了。


# push 的时候出现 change-id message

可以根据命令行中给出的提示
```
 gitdir=$(git rev-parse --git-dir); scp -p -P 29418 jieyu.chen@gerrit.17zuoye.net:hooks/commit-msg ${gitdir}/hooks/
```

执行命令

然后如果是最后一次提交，则执行`git commit --amend` 在打开的 vim 编辑器中不做任何修改即可，直接`:wq`退出，然后再次查看 `git log` 看看是否补上了。


https://blog.csdn.net/liuxu0703/article/details/54343096


gitdir=$(git rev-parse --git-dir); scp -p -P 29418 jieyu.chen@gerrit.17zuoye.net:hooks/commit-msg ${gitdir}/hooks/

## 如果之前有多个提交没有 change-id 的解决办法：

首先还是像上面说的那样执行提示给出的命令，然后

1、git branch work（从最新节点建立分支，相当于将自己的修改备份到新的分支）

2、git reset --hard HEAD~10（强制回滚多个节点）

3、git status 

如果显示nothing to commit, working directory clean，跳到5.

如果显示has x commit，xx git push 跳到2.

4、git clean -df

5、git pull（使得当前分支和线上统一）

6、git merge --squash work（将最开始建立的分支中的内容合并回来）

7、git commit

8、git push


# 标签

## 打标签

> git tag -a vx.x.x -m "xxxxx"

也可以不用 -m 以及内容，但是在 gerrit 上有时候没有注释的话标签不能推送到服务器，所以最好还是加上

## 查看标签

> git tag

## 补充标签

比如某次上线后忘了打标签，可以针对某次 commit 来打标签

> git tag -a vx.x.x changeID -m "xxx"

## 推送标签

推送所有标签

> git push origin --tags  

推送某个标签

> git push origin vx.x