# 操作指令

## remote

```javascript
git remote add xxx(别名) xxxxxxx(远程仓库地址)  // 如果是一个新创建还没有.git 文件夹的新文件夹需要进行初始化
git remote set-url xxx(别名) xxxxxxx(远程仓库地址)  // 改变连接位置
例如：git remote add origin 1.1.1.1:/home/zzh/repository/flutter.git
git remote -v  // 查看已经关联的远程仓库
git remote remove xxx(远程仓库别名)  // 删除已关联的远程仓库
```
## init

```javascript
git init  // 通常在新建文件夹之后
```
## push一套

* add将工作区的修改添加到暂存区
* commit将暂存区修改提交到本地仓库
* push将本地仓库推送至远程仓库
```javascript
git add file1.txt  // 添加单个文件
git add directory/  // 添加整个文件夹
git add directory/abc.java  // add和.git同级的某个文件夹下的文件windows上也是这么写路径
git add *.txt  // 添加所有 txt 文件
git add -p  // 交互式模式，可以选择性地为文件索引修补
git add *  // 将将工作区所有修改的文件都添加到暂存区
git add -u  // 将已被跟踪的文件的修改添加到暂存区，而不会添加新文件
git commmit -m"some infomation"
git reset HEAD~  // 撤销最近一次commit，但是保留工作区改动
git push <远程仓库名称> <本地分支>:<远程分支>
git push b abranch:bbranch  // 实现在当前a仓库的abranch推送到b仓库的bbranch
git push <远程仓库名称> --all  // 将本地的所有branch一起push
git push origin master
git push --set-upstream <origin> <newBranch>  // 上传新建立的分支
```
## status

```sql
git status  // 查看当前工作区和暂存区的状态。
```
## 连接ubuntu电脑、pull、连接仓库

```javascript
ssh <username>@1.1.1.1 "cd /home/zzh/repository && mkdir flutter.git && cd flutter.git && git init --bare"  // 给远程的linux系统创建git仓库
git pull <远程仓库地址> <远程分支名称>
git pull <远程仓库地址>
```
## show、log

```javascript
git show origin/master   // 输出远程 master 分支最新的一次 commit 信息并显示其更改的内容
git log <branchName>  //查看某一分支的commit情况
git log  // 查看当前分支的commit情况
git log --name-status HEAD^..HEAD  // 显示最近一次提交的详细信息，包括修改的文件列表和状态。其中，HEAD^..HEAD表示最近一次提交的范围。显示多少个文件就代表这次上传了多少的文件。比如M       oboqrcs/oboqrc.log代表只上传了oboqrc.log且是对这个文件进行了修改操作。
```
## branch

```javascript
git branch                  // 查看分支
git branch -a               // 查看远程和本地所有分支
git branch xxx              // 本地创建 xxx 分支（内容与当前分支保持一致）
git branch -d xxx(分支名)    // 删除 xxx 分支
```
## checkout

```javascript
git checkout xxx(分支名)     // 切换至 xxx 分支
git checkout -b xxx(分支名)  // 创建并切换至 xxx 分支
git checkout -f  // 丢弃工作区内容， 注意： 该操作会删除掉所有本地工作区修改的所有内容并回到未修改前的状态，请确保自己是否需要该操作
```
## merge

```javascript
git merge xxx(需要合并到当前分支上的其他分支名)
git merge --abort  // 合并代码没有 add 时可使用该命令取消 merge，如果代码已经 add，则需要先回退操作之后再取消 merge
```
## stash

```javascript
git stash
git checkout linuxVersion
git stash pop
git stash list  // 查看stash里面是否有内容
git stash drop  // 删除最近一次stash
// 第一行命令将您所有的未提交更改放入 stash 中。然后，您可以切换到 linuxVersion 分支，并执行任何您需要连接此分支的操作。最后，第三行命令将之前存储的更改从 stash 中取出，并将其应用于您当前所在的分支。
```
## fetch、ls-tree

```javascript
git fetch origin main  // 以获取最新的远程仓库提交记录和快照信息
git fetch -v  // 查看连接的那个仓库
git ls-tree origin/main  // 查看某分支的文件目录结构
```
## reset、revert

```javascript
git reset --hard <commit hash>  // 切换到指定的commit,reset就会删除没有push的commit了，如果已经push则还需要加上revert
git revert <commit hash>  // 删除某个commit，但是你不能站在当前commit删除当前commit
```
## cherry-pick

```javascript
// 多branch进行某一部分同步时可以使用
// 作用：应用其他某branch的某次commit
假如zzh.java进行了修改并且这个修改需要同步到其他分支
git add directory/zzh.java
git commit -m"..."
git log  // 保存刚才的commit-hash
git checkout 想同步的branch
git cherry-pick <commit-hash>
```
# 技巧

## 创建新版本

1. 只有一份源代码但是里面有多个分支，切换分支进行代码修改，它们是独立的。比如只有一份源代码，master分支里面zzh.vue里面的内容是masterInfo，切换至newBranch1修改zzh.vue里面的内容为newBranchInfo。push后此时你查看master里面还是masterInfo、newBranch1里面还是newBranchInfo
2. 实现上面的情况有个前提，就是newBranch1必须得commit不然就算切换回master你看到的还是newBranch1里面的内容
## 文件被锁定

```javascript
Unlink of file 'oboqrcs/oboqrc.log' failed. Should I try again? (y/n) n
error: unable to unlink old 'oboqrcs/oboqrc.log': Invalid argument
fatal: Could not reset index file to revision 'HEAD'.
```
oboqrc.log文件被某进程锁定、或者没有足够权限修改它。很明显这里是后端没关闭后端将它锁定了。
## git hooks

* commit前使用eslint检查代码格式
```javascript
// eslint安装
npm install eslint --save-dev
// package.json的script中进行配置
"lint:eslint": "eslint --fix --ext .js,.ts,.vue ./src",
"lint:prettier": "prettier --write --loglevel warn \"src/**/*.{js,ts,json,tsx,css,less,scss,vue,html,md}\"",
"lint:stylelint": "stylelint --cache --fix \"**/*.{vue,less,postcss,css,scss}\" --cache --cache-location node_modules/.cache/stylelint/",
"lint:lint-staged": "lint-staged",
"prepare": "husky install",
"release": "standard-version",
"commit": "git status && git add -A && git-cz"
// 安装
npm install husky lint-staged --save-dev
pnpm i husky -D
// 根目录对husky初始化
npx husky install
// 生成pre-commit文件
npx husky add .husky/pre-commit 'npm run lint:eslint'
```
### merging

处于(yihuaVersion|MERGING)这种合并状态，并解决完冲突后，重新使用git add、commit、push这一套结束merging状态

## 同时连接多个远程仓库

```typescript
git remote add main git@1.1.1.1:zzh/flutter.git  // 使用remote add去新连接其他仓库，但是本地的仓库别名origin、main唯一
git push b abranch:bbranch  // 实现在当前a仓库的abranch推送到b仓库的bbranch
git push <远程仓库名称> --all  // 将本地的所有branch一起push，这个远程仓库名就是刚才add或者set-url的仓库别名
```
## 处理上传错误的文件

```typescript
git rm hs_err*.log  // 删除满足条件的文件，*是通配符
git commit -m
git push
```
# GIT 团队合作

## 方式一：克隆 git clone

1、克隆远程仓库到当前目录，（连同文件夹一起克隆）文件夹名称与远程仓库文件夹名称相同

```xml
git clone xxx（远程仓库地址）
```
2、查看远程分支
克隆成功后，本地仓库只有 master 一个分支与远程关联，master 为主分支，一般我们不会再该分支上开发，所以我们要切换到开发分支上，此时我们需要查看远程的分支

```javascript
git branch -a  // 查看远程和本地所有的分支
```
3、切换到本地的开发分支或者新建一个newBranch
```xml
git checkout dev 
git checkout -b newBranch
```
如果远程分支有 dev 分支，那么该操作会在本地生成一个与之关联的 dev 分支 并且切换至本地 dev 分支
## 方式二： 拉取 git init->git remote

1、创建仓库

```javascript
git init  // 创建仓库
```
    生成本地 master 主分支 （此时分支内容为空）
2、关联远程仓库

```javascript
git remote add origin xxx    // 与远程仓库关联，并起别名为 origin
git remote -v                // 查看关联的远程仓库
```
3、 拉取远程仓库内容并与本地 master 产生关联 
```javascript
git fetch orign          // 拉取远程仓库分支内容
git pull origin master   // 获取远程仓库 master 分支内容并与本地 master 建立关联关系
```
5、切换远程分支并创建与之关联的本地分支
```javascript
git branch -a             // 查看远程和本地所有的分支
git checkout xxx          // 切换至 xxx 分支（创建本地分支并与远程仓库分支产生关联）
```

