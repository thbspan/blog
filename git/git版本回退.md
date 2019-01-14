# git版本回退

## git reset 修改head指针的位置，不会产生新的提交
    注意：reset --soft --hard --mixed --merge等参数的含义可以参考官方文档
    这样回退会存在一些问题
### 问题1、如果代码推送到的远程仓库，需要执行git push -f强制推送，因为本地的HEAD指针位置在后面）。

### 问题2、会清除之前的提交日志（本地可以使用git reflog查看）

### 问题3、团队的其他人需要手动用远程master分支覆盖本地master分支。如果其他成员看到本地版本比较新直接进行了推送，那么你的回滚就会被还原

### 用法说明
    git revert HEAD //撤销最近一次提交，由于是HEAD --mixed，暂存区的目录树会被重写，被 master 分支指向的目录树所替换，但是工作区不受影响
    git revert HEAD~1 (HEAD~2 HEAD~n) // HEAD之前的N个版本
    
## 更方面的方式git revert
    git revert HEAD //撤销最近一次提交
    git revert HEAD~1                   //撤销上上次的提交，注意：数字从0开始
    git revert 0ffaacc                  //撤销0ffaacc这次提交
    注意：
    
### 说明
    git revert 命令意思是撤销某次提交。它会产生一个新的提交，虽然代码回退了，但是版本依然是向前的，所以，当你用revert回退之后，所有人pull之后，他们的代码也自动的回退了。
### 注意点1
    只是撤销一次提交
    如果要撤销多次提交
    git revert OLDER_COMMIT^..NEWER_COMMIT
    如果我们想把这三个revert不自动生成三个新的commit，而是用一个commit完成，可以这样
    git revert -n OLDER_COMMIT^..NEWER_COMMIT
    
### 注意点2
    使用revert HEAD是撤销最近的一次提交，如果你最近一次提交是用revert命令产生的，那么你再执行一次，就相当于撤销了上次的撤销操作，换句话说，你连续执行两次revert HEAD命令，就跟没执行是一样的
    
### 注意点3：如果使用 revert 撤销的不是最近一次提交，那么一定会有代码冲突，需要你合并代码，合并代码只需要把当前的代码全部去掉，保留之前版本的代码就可以了.

### 注意点4：如果分支合并后需要revert，需要指定-m parent-number参数
    -m 后面带的参数值,对应着parent的顺序，从1开始，可以根据log图形日志查看.参考文档 https://segmentfault.com/a/1190000015792394 理解
    
    
