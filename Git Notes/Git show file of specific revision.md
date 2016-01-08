### 查看旧版本文件
有时候想查看某次commit的某个文件内容，单纯用git show会显示commit的diff，此时要
> git show ref:file

这样就可以查看完整内容

###技巧

* 用vim来编辑
> git show ref:file | vim -

* 用vim编辑且带语法
> git show ref:hello.rb | vim -c 'set syntax ruby' -

### Reference
1. [View a file in a different git branch without changing branches](http://stackoverflow.com/questions/7856416/view-a-file-in-a-different-git-branch-without-changing-branches)
2. [How to retrieve a single file from specific revision in Git?] (http://stackoverflow.com/questions/610208/how-to-retrieve-a-single-file-from-specific-revision-in-git/610315#610315)
