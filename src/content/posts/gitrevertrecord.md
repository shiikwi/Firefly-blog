---
title: 记录一次git revert与reset的使用
published: 2026-06-26
description: git revert和reset使用记录
tags: [git, 芝士]
category: 芝士
slug: git-revert-record
draft: false
---
git的撤销虽然之前在搓开源小玩具的时候用到过，但是实际开发中遇到的情况要复杂不少。
![](https://img.koisukui.top/file/1782621964933_2.png)
简单说明一下，4908dad9进行了一个模块移除的操作，但是没处理好，所以f8dba915对它进行了revert。  
发现没有完全恢复后，进行了误操作，又revert了一次，871f9300这次revert误删了很多无关的文件。所以我只能对这个revert再进行revert。  
这样其实就相当于这四个commit啥都没干，既没有完成模块的删除，也没有因为误操作删除多余文件。
# git reset
先说git reset，由于误操作导致git history已经引入了太多无用commit，为了方便后面举例，先用reset清理一下。  
reset可以撤销记录到某个指定的commit，不生成新增commit。  
reset一共有三种模式
```bash
git reset --soft HEAD~1 # 仅移动HEAD指针到目标提交
git reset --mixed HEAD~1 # 移动HEAD指针到目标提交，重置暂存区
git reset --hard HEAD~1 # 移动HEAD指针到目标提交，重置暂存区与工作区
```
把提交reset到9becf182这个merge commit，所以应该把HEAD指针往下移4个
```bash
git reset --soft HEAD~4
```
![](https://img.koisukui.top/file/1782621961871_1.png)
可以看到本地已经回到了我们想要的位置，然后同步云端，直接强推覆盖(这是我fork的仓库，实际最好不要这么干，可能会被骂死。。。)
```bash
git push --force
```

# git revert
revert可以撤销某几次commit，同时生成一个新的commit记录这次撤销操作。  
语法如下
```bash
# 撤销提交
git revert commit-hash

# 撤销多次提交，不创建commit
git revert -n commit-hash1 commit-hash2

# 撤销merge
git revert -m 1 merge_commit_hash
```
我们的目标是以c44f60f8这个commit作为起点，保留e9b1782f 1aa3c7c8 c94ef88333 6d833a9c这四个commit，撤销其它的。  
这里应该用的是第二个，由于需要处理的commit路径里有两个merge，同时需要保留分支中的commit，所以不适合直接批量撤销。  
分析一下，实际我创建的分支功能全部需要保留，需要撤销的commit均位于主分支上，那么其实就只需要
```bash
git revert -n 74581892 3959f689 497255ed 9683c54d
```
merge commit无需额外处理，因为父提交为9becf182的commit全部需要保留。如果有冲突，解决冲突后commit即可。这样就会生成一个commit来记录这次revert。  
如果提交是线性的，位于一条主线上，可以用下面的命令批量操作。
```bash
git revert old_commit..new_commit #不包括old_commit
```