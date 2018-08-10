### adb 命令查看 包名 对应包名 apk

* 查看手机上应用的packageName

  ```
  adb shell pm list packages 
  ```

* 查看应用对应的apk文件在手机上的安装位置

  ```
  adb shell pm list packages -f 
  ```

* 根据某个关键字查找包 

  ```
  adb shell pm list packages | grep 关键字
  ```





### 根据包名启动应用

```
adb shell am start -n 包名/完整类名
```



### 得到system读写权限 

```
adb shell
mount 查看system目录
mount -o remount /dev/block/by-name/system
```



### 查看当前运行的进程

```
adb shell ps
```



