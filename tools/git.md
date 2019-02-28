# Git

假设，你想写一个网站，已经建了个叫做site-fe的文件夹用于存储前台页面，你命令行进入了这个文件夹，并想要用git管理你的网站版本，将代码托管到github，接下来你要怎么做？

如果你已经安装git, 在当前文件夹命令行输入`git init`回车（在当前目录创建一个名为 .git 的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件）后便会将当前文件夹初始化为git仓库。然后你就可以用git add/git commit等命令搭配参数对当前仓库进行本地版本管理。<br>

## 如何将代码推到github

首先你要有一个github账号，然后新建repository，新建完后会提示你给本地项目添加远程仓库，有https和ssh两种通讯协议，两种方式都可以设置免密传输，我通常是选择ssh。本地命令行运行`git remote add origin git@github.com:<userName>/<repoName>.git` 就给本地仓库添加了远程仓库，`git remote -v`可以查看。如果要修改可以输入`git remote set-url origin <new remote>`。然后运行git push就可以把本地commit了的代码提交到远程了。

## 如何使用ssh免密推拉代码

你可以在github个人主页setting/ssh里面将电脑命令行ssh-keygen默认生成的`~/.ssh/id_rsa.pub`文件内容提交，由于id_rsa.pub这个文件内容是唯一的，这样就相当于绑定了这台电脑与github账户，以后这台电脑ssh方式拉取和推代码就不需要输入用户名和密码。

**如果你又开了一个github账户，还是想用ssh形式推送代码，但是在第二个账户添加本电脑ssh公钥的时候提示公钥已经被使用，你该怎么办？**

这时候你可以用ssh-keygen生成一对与之前不同名的新的密钥，因为执行这个命令的时候会提示你输入密钥名称。进行ssh传输的时候系统默认会使用id_rsa这个名称的文件，自定义名称后系统又如何知道你使用的是哪个密钥呢？在`~/.ssh/`中你可以新建一个叫config的文件，如下配置：

```
Host test_github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/<新密钥名称>
```
然后，将代码仓库的远程仓库地址设置成`git@test_github.com:<userName>/<repoName>.git`。

## 常用的git命令

```
git status // 查看仓库状态
git add .
git commit -m <提交注释>
git fetch // 将远程版本库的更新取回到本地
git pull // 拉取远程代码
git push origin <分支名> // 推代码到远程
git checkout -- . // 丢弃对所有文件的更改
git checkout -b <new branch> // 在当前分支基础上新建并切换到新分支
git checkout <branch> // 切换到新分支
git branch [-a|r] // 查看当前/所有/远程分支
git log // 查看本地提交记录
git remote -v // 查看远程仓库地址
git add remote origin <仓库地址> // 添加远程仓库
git remote set-url origin <新仓库地址> // 设置远程仓库地址
git stash // 将本地修改储存起来，代码恢复到修改前
git stash pop // 将上次stash的代码恢复出来
git reset --hard <commitId> // 将本地代码强制恢复到某个commit
git add ./ fix conflict / git rebase --continue
git diff <commitId> // 对比当前与某次提交代码到差异，也可比较任意两次commit的差异
```

命令行输入`git status`后能查看仓库中代码状态，有的时候你不想push当前文件夹中的一些文件，比如记录了api私钥信息的文件，或者node_modules这种没必要上传的文件夹，这个时候你可以在当前目录新建一个.gitignore文件，然后将你想忽略上传的文件或文件夹通过换行隔开记录在里面。如下：

```
node_modules
secret_file.js
```


