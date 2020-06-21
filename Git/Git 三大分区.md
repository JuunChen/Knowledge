![image-20200603160155566](/Users/momo/Library/Application Support/typora-user-images/image-20200603160155566.png)

### working directory (工作目录)

它是工作目录，是我们肉眼能够看到的文件。

在工作目录执行 `git add`后，就会把工作目录中的**<font color=#8e2323>修改</font>**添加到暂存区。

### stage area (index area) (暂存区)

当暂存区存在修改时，我们使用 `git commit`命令，就会把暂存区中的**<font color=#8e2323>修改</font>**提交到 history区 中。

### commit history (history区)

只要不乱动本地的`.git`文件夹，进入`history`的**<font color=#8e2323>修改</font>**就永远不会丢失。 



每一个 commit 都已一个唯一的 hash 值，我们经常说的 HEAD 或者 master 分支，都可以理解为一个指向某个 commit 的指针。

work dir 和 stage 区域的状态，可以使用 git status 来查看。

history 区域的提交历史可以通过 git log 命令来查看。



> 小结：在工作区域产生修改，使用 git add 将修改添加到暂存区，使用 git commit 将暂存区的修改提交到history区。
> 每次commit的内容都是修改，都有唯一的hash值。

