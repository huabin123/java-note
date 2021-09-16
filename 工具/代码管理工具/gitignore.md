### Git忽略规则.gitignore不生效

#### 场景

添加gitignore-->添加不在gitignore中被忽略提交的文件-->无法忽略新文件

#### 原因

 .gitignore 只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。

#### 解决方法

先把本地缓存删除（改变成未track状态），然后再提交。

```shell
git rm -r --cached .

git add .

git commit -m 'update .gitignore'
```



