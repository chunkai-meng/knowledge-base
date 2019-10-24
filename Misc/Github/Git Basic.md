# Git Basic

## 1. 安装
### 添加SSH Key
[官方介绍](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#)


### Push an existing repository from the command line  
```
git remote add origin https://github.com/chunkai-meng/symfony.git
git push -u origin master
```

```
git config credential.helper store
```

## 2. 使用
### Permanently ignore changes to a file

If a file is already tracked by Git, adding that file to your .gitignore is not enough to ignore changes to the file. You also need to remove the information about the file from Git's index:

These steps will not delete the file from your system. They just tell Git to ignore future updates to the file.

Add the file in your .gitignore.

Run the following:
```
> git rm --cached <file>
```
Commit the removal of the file and the updated .gitignore to your repo.

## 3. 技法
### 书写Commit Message的原则
1. 首行是少于50字标题，首字母大写，末尾没有句号.
2. ~/.vimrc 加入
```
autocmd Filetype gitcommit setlocal spell textwidth=72
```
3. TITLE：标题用命令式语气，表明采用了本次commit以后会发生什么（e.g. "Add auth to X"）
4. 留空一行写Body，用小于72个字符，描述*what* and *why* vs. *how*。
5. BODY：解释这次 commit 的目的（产生的影响，甚至副作用）。至于how（如何产生）不重要，代码已经解释了


### 4. 忽略文件
```
#               表示此为注释,将被Git忽略
*.a             表示忽略所有 .a 结尾的文件
!lib.a          表示但lib.a除外
/TODO           表示仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/          表示忽略 build/目录下的所有文件，过滤整个build文件夹；
doc/*.txt       表示会忽略doc/notes.txt但不包括 doc/server/arch.txt
 
bin/:           表示忽略当前路径下的bin文件夹，该文件夹下的所有内容都会被忽略，不忽略 bin 文件
/bin:           表示忽略根目录下的bin文件
/*.c:           表示忽略cat.c，不忽略 build/cat.c
debug/*.obj:    表示忽略debug/io.obj，不忽略 debug/common/io.obj和tools/debug/io.obj
**/foo:         表示忽略/foo,a/foo,a/b/foo等
a/**/b:         表示忽略a/b, a/x/b,a/x/y/b等
!/bin/run.sh    表示不忽略bin目录下的run.sh文件
*.log:          表示忽略所有 .log 文件
config.php:     表示忽略当前路径的 config.php 文件
 
/mtk/           表示过滤整个文件夹
*.zip           表示过滤所有.zip文件
/mtk/do.c       表示过滤某个具体文件
```

被过滤掉的文件就不会出现在git仓库中（gitlab或github）了，当然本地库中还有，只是push的时候不会上传。

需要注意的是，gitignore还可以指定要将哪些文件添加到版本管理中，如下：
!*.zip
!/mtk/one.txt

唯一的区别就是规则开头多了一个感叹号，Git会将满足这类规则的文件添加到版本管理中。为什么要有两种规则呢？
想象一个场景：假如我们只需要管理/mtk/目录中的one.txt文件，这个目录中的其他文件都不需要管理，那么.gitignore规则应写为：：
/mtk/*
!/mtk/one.txt

假设我们只有过滤规则，而没有添加规则，那么我们就需要把/mtk/目录下除了one.txt以外的所有文件都写出来！
注意上面的/mtk/*不能写为/mtk/，否则父目录被前面的规则排除掉了，one.txt文件虽然加了!过滤规则，也不会生效！

还有一些规则如下：
fd1/*
说明：忽略目录 fd1 下的全部内容；注意，不管是根目录下的 /fd1/ 目录，还是某个子目录 /child/fd1/ 目录，都会被忽略；

/fd1/*
说明：忽略根目录下的 /fd1/ 目录的全部内容；

/*
!.gitignore
!/fw/ 
/fw/*
!/fw/bin/
!/fw/sf/
说明：忽略全部内容，但是不忽略 .gitignore 文件、根目录下的 /fw/bin/ 和 /fw/sf/ 目录；注意要先对bin/的父目录使用!规则，使其不被排除。


5. 上传代码到独立分支
一、上传一个独立的分支（比如代码是从工程中直接DOWNLOAD ZIP文件如BowlingScore-test.zip，该文件与原MASTER分支是独立的）

1、Git init （在本地工程目录下），生成.git 文件夹

```
Git init
```

2、上传修改的文件

```
git add *
```

(*可替换成具体要上传的文件名，*表示提交所有有变化的文件) 3、添加上传文件的描述

```
git commit -m "test" 
```
 （”test“为分支名）

4、（创建分支）

```
git branch test
```

5、（切换分支）
```
git checkout test
```

6、与远程分支相关联
```
git remote add origin https://github.com/yangxiaoyan20/BowlingScore.git
```
   （”BowlingScore“ 为工程名）
7、（将分支上传）
```
git push origin test
```