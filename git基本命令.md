#### Branch

##### 删除远程分支

```
git push origin --delete branchname
```

##### clone某个分支到本地

```
git clone -b <分支名> git@adc.github.trendmicro.com:Commercial-DDEC/uiserver.git
```

##### 将commit push到github上

```
git push origin feature/DDEC-**
```

##### 在本地将dev merge到自己的分支

```python
# 先换到dev上，把本地代码更新到最新，再切换到当前分支 
git checkout -b develop origin/develop
git pull
git checkout feature/ddec-password_analyzer
git merge dev 
```



#### 撤销修改

##### 1. 未add到暂存区

###### 放弃修改

单个文件/文件夹

```
git checkout -- filename
```

所有文件/文件夹

```
git checkout .
```

###### 放弃新增

单个文件/文件夹

```
rm filename / rm dir -rf
```

所有文件/文件夹

```
git clean -xdf
```

放弃本地所有修改

```
git reset --hard HEAD^
```

##### 2. 已经add到暂存区

单个文件/文件夹

```
git reset HEAD filename
```

所有文件/文件夹

```
git reset HEAD .
```

##### 3. 已经commit

撤销之后，commit的修改在工作区

```
git reset commit_id
```

撤销之后，工作区/暂存区/commit的修改都会清除

```
git reset -- hard commit_id
```

##### 4. 已经push