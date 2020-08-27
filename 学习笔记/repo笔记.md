#repo学习笔记
##安装流程
###1安装Git
###2安装python3
###3安装repo
####3.1下载repo
打开git base执行以下代码
```
mkdir ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+rx ~/bin/repo
```
把C:\Users\你的用户名\bin添加到环境变量
####3.1下载repo工具源代码（不确定是不是必要步骤，待验证）
```
# 先随便新建源码目录
mkdir -p ~/AOSP/.repo
cd ~/AOSP/.repo
# clone工具集
git clone https://gerrit.googlesource.com/git-repo
# 一定要改文件夹名
mv git-repo repo
# 回到AOSP源码目录
cd ..
```
###4初始化manifest
####4.1代码库新建manifest项目
####4.2新建default.xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <remote  name="repoTest"
           alias="origin"
           fetch="https://github.com/" />
  <project path="xyrxizhi/mytestgit" name="xyrxizhi/mytestgit" groups="null" revision="master" remote="repoTest"/>

<!-- ... -->
</manifest>
```
每个属性代表的含义占坑
####4.3初始化manifest
在项目本地地址执行
```
repo init -u 你的manifest项目链接
# 比如android的项目
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-10.0.0_r25 --worktree
```
###下载远端代码
```
repo sync //-j4多线程下载
```
##参考文献
```
https://juejin.im/post/6844903718316408840
https://juejin.im/post/6844904057421742094
```