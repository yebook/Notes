## Git使用记录

### git基础使用

```
##创建本地仓库
git init  
##添加文件
git add xxx.md 
##进行提交
git commit -m "first commit" 
##关联远程仓库
git remote add origin git@xxx.xxx.xxx.xxxx:sn/xxx.git 
##移除远程仓库
git remote rm 远程仓库别名 
##推送代码至远程库的master分支
git push -u origin master 

===删除文件======
1.在本地删除文件
2.执行 git rm * -r（记得，cd 到你要删除的目录下。当然 * 可以换成指定目录,删除单个文件git rm <文件>）
3.git add .
4.commit 和 push
```



