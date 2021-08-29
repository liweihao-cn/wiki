# Git

## 常用操作

### 代码合并

* merge
* rebase

### 摘取提交

* cherry-pick

### 修改某个commit

```shell
# 修改之前的某个提交，commit-id为想修改的提交之前的那个提交
git rebase -i commit-id
```

### 查看某行代码的提交记录

```shell
# 查看某行代码的提交记录
git blame -L line
```

### add某一行或者某几行

```shell
git add -p
```

或者使用tig，直接在Unstaged changes中的某一行按1可以切换这一行的状态
