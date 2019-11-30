### 参考

https://networm.me/2018/05/13/migrate-to-gitlfs/

 

### 第一步，准备环境

由于lfs服务器与仓库地址不一致，需要先设置一下lfs服务器

```
git config --global lfs.url http://acountXXX:passwordXXX@jfrog.cloud.qiyi.domain/api/lfs/iqiyi-gitlfs-vrasi/
git config --global lfs.concurrenttransfers 8
```

 

### 第二步，下载版本

```
git clone --mirror http://gitlab.qiyi.domain/vrasi/robot_ue4.git
```

![](https://raw.githubusercontent.com/badforlabor/rawimage/master/20191130111251.png)

 

 

### 第三步，转化大文件

下载git-lfs-migrate.jar，链接是：https://github.com/bozaro/git-lfs-migrate/releases/latest

然后执行下面命令进行转化即可

```
// -s 源仓库 我的是 robot_ue4.git
// -d 指定转化后的目标仓库目录，会自动创建此目录。我的是 robot_ue4_converted.git
// -l 指定lfs-server，必须得带上gitlab账号密码，我的是 http://accoutXXX:passwordXXX@jfrog.cloud.qiyi.domain/api/lfs/iqiyi-gitlfs-vrasi/
// 最后一堆参数是大文件的类型，是blob格式，区分大小写。
java -jar d:\bfg\git-lfs-migrate\git-lfs-migrate.jar -s robot_ue4.git -d robot_ue4_converted.git -l http://accoutXXX:passwordXXX@jfrog.cloud.qiyi.domain/api/lfs/iqiyi-gitlfs-vrasi/ --write-threads 8 "*.jpg" "*.mb" "*.mov" "*.mp4" "*.avi" "*.[Ff][Bb][Xx]" "*.abc" "*.png" "*.[Tt][Gg][Aa]" "*.umap" "*.uasset" "*.exe" "*.dll" "*.pdb" "*.ztl" "*.ZTL" "*.psd" "*.spp" "*.rar" "*.zip" "*.wav" "*.wwu" "*.wem" "*.bnk" "*.docx" "*.xls" "*.exr" "*.unity" "*.tif" "*.unitypackage"
 
```

![](https://raw.githubusercontent.com/badforlabor/rawimage/master/20191130112020.png)

 

注意，执行成功后，需要检查一下转化后的仓库大小是否小于2G，如果大于2G，说明肯定还有大文件没有转移到lfs-server上。

检索都有哪些大文件，可以使用这个小工具，http://gitlab.qiyi.domain/liubo04/folder

![](https://raw.githubusercontent.com/badforlabor/rawimage/master/20191130104321.png)

 

 

### 第四步，提交仓库

先在gitlab上，创建一个名为robot_ue4_converted的仓库，然后将转化后的仓库提交上去。

```
cd robot_ue4_converted.git
 
// 检查是否仓库完整。我发现，转化了十几个仓库，经过检查都没有问题，还没有失败过，所以这个命令可以忽略
git fsck
 
git push --mirror http://gitlab.qiyi.domain/vrasi/robot_ue4_converted.git
```

![](https://raw.githubusercontent.com/badforlabor/rawimage/master/20191130160513.png)

 

 

### 第五步，将lfs.url配置到当前工程中

准备一份.lfsconfig文件，内容是

```
[lfs]
    url = http://jfrog.cloud.qiyi.domain/api/lfs/iqiyi-gitlfs-vrasi/
```

 

由于上一步转化完毕后，并没有将lfs.url写入到.lfsconfig中，所以需要手动更改下。（如果能在上一步完成是最完美的）

```
git clone http://gitlab.qiyi.domain/vrasi/robot_ue4_converted.git
cd robot_ue4_converted
//copy ..\.lfsconfig .\
git add .lfsconfig
git commit -m "迁移大文件"
git push
```

![](https://raw.githubusercontent.com/badforlabor/rawimage/master/20191130162258.png)

 

 

### 第六步，将lfs.url配置到其他分支中

如果存在很多分支，一定也要把各个分支都设置一下。

```
git branch -a
git checkout -b branchXXX origin/branchXXX
//copy ..\.lfsconfig .\
git add .lfsconfig
git commit -m "迁移大文件"
git push --all origin
```

 

 

### 第七步，验证

到此步，其实迁移已经完成了，但是为了确保正确，需要验证一下。

可以去别人的机器上clone仓库验证，也可以在自己机器上验证。

如果在自己机器上验证，需要清理一下环境，否则会干扰验证结果。

**<font color="#ff0000">注意，一定要用git lfs clone</font>**

```
git config --global --unset lfs.url
git lfs clone http://gitlab.qiyi.domain/vrasi/robot_ue4_converted.git
```

如果发现**仓库下载成功，并且历史记录都在**，就说明没有问题了。

额外，如果已经用git clone下载仓库，那么需要额外在加一条命令 git lfs fetch --all

```
git clone http://gitlab.qiyi.domain/vrasi/robot_ue4_converted.git
cd robot_ue4_converted
git lfs fetch --all
```

 

 

### 第八步，清理环境

最终，所有迁移完毕后，去掉全局设置的lfs.url

```
git config --global --unset lfs.url
```

 

 

### 第九步，在gitlab网站上操纵

在gitlab上，将原有仓库设置成无法访问（或者archived）

![](https://raw.githubusercontent.com/badforlabor/rawimage/master/20191130161908.png)

另外，将原有仓库设置成readonly：http://gitlab.qiyi.domain/vrasi/robot_ue4_readonly

将converted仓库，设置成正常的：http://gitlab.qiyi.domain/vrasi/robot_ue4