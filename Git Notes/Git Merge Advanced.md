###重新merge

merge产生冲突，会标记成conflict，一旦对冲突文件使用add，即表示冲突解决了。如果只是不小心add或者没解决完冲突就标记成解决了，如果还没提交，这时想重新解决冲突：

> git checkout --conflict=merge file

重新标记冲突，默认是merge，改成diff3能显示base。

**重要**:一定要加上文件，不然就变成单纯的checkout head，让working space/index 与 head同步 ，但自动merge，这样看起来任何变化都没有，但原来的 `All conflicts fixed but you are still merging.` 信息会丢失


###高级merge

[高级合并](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%AB%98%E7%BA%A7%E5%90%88%E5%B9%B6)

**解决冲突时：**
> git show :1:file > file.common
>
> git show :2:file > file.ours
>
> git show :3:file > file.theirs

可以用`git ls-files -u` 来查看

用`git merge-files -p file.ours file.common file.theirs > file`  来三路合并

**解决冲突但还没用add表示解决，作最后确认时：**
> git diff --ours  查看自己与解决好冲突的文件的不同，看引入什么
> 
> git diff --theirs 查看merge进来的文件与解决好冲突的文件的不同，看哪些是引入进去他们的文件
>
> git diff --base 查看共同分歧点的文件与解决好冲突的文件的不同，看解决文件都引入些什么改动
