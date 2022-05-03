# git基本使用

```bash
git init
```

git init  初始化git的管理的项目，创建暂存区



``` bash
git clone  [url]   
```

git clone  [url]   克隆git上面的项目



## 版本管理

``` bash
git status
```

git status 查看我文件状态



``` bash
git add .
```

git add .  把文件夹下所有的文件添加到git暂存区，在git根文件夹里添加 **.gitignore**用来忽略那些文件不被添加到git暂存区里



``` bash
git commit -m""
```

git commit -m""  提交信息加入暂存区所备注的信息



``` bash
git reflog
```

git reflog 用来查看精简版的版本信息



``` bash
git log
```

git log 查看全部版本信息



``` bash
git reset [ --soft | --mixed | --hard] [7位版本号]
```

git reset [ --soft | --mixed | --hard]  [7位版本号]   

- **--mixed** 为默认，可以不用带该参数，用于重置暂存区的文件与上一次的提交(commit)保持一致，工作区文件内容保持不变。
- **--soft** 参数用于回退到某个版本。
- **--hard** 参数撤销工作区中所有未提交的修改内容，将暂存区与工作区都回到上一次版本，并删除之前的所有信息提交。**谨慎使用**



## 分支管理

``` bash
git branch -v
```

git branch -v 查看所有分支



``` bash
git branch (分支名)
```

git branch [分支名]   创建分支



``` bash
git checkout (分支名)
```

git checkout (分支名)  跳转分支



``` bas
git switch (分支名)
```

git switch (分支名)  切换分支   **git 2.23 版本新增**



``` bash
git merge (要合并的分支名)
```

git merge (要合并的分支名)  合并分支



## 远程版本库

``` bash
git remote add [别名] [url]
```

git remote add [别名] [url]  添加远程版本库



```bash
git remote -v 
```

git remote -v  查看远程版本库



``` bash 
git push [远程版本库别名] [要提交的分支名]
```

git push [远程版本库别名] [要提交的分支名]   把代码提交到远程版本库



``` bash
git pull [远程主机名] [远程分支名]:[本地分支名]
```

git pull [远程主机名] [远程分支名]:[本地分支名]   取回远程主机某个分支的更新，再与本地的指定分支合并