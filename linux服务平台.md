# linux 服务平台

## 写在前面的话

- 主要总结自己几年来，维护办公室用于机器学习的服务器的一些心得。办公室服务器用的是Ubuntu16.4LTS。

- 内容顺序根据使用顺序排列



## 目录

[-toc] 

   

## 系统管理

### 安装ubuntu

- 使用U盘安装，网上很多教程，目前用ubuntu16.4,LTS  和18.4LTS的居多
- 目前服务器都是固态（小储量）+机械硬盘（大储量），**安装系统到固态硬盘上，一定要把机械硬盘挂载在` /home` 文件目录上**，不然默认的是在固态硬盘上，这出现硬盘的存储空间不足的问题。
- 忘记把机械硬盘挂载在` /home` 也可以手动迁移/home  上的文件到机械硬盘，然后再完成挂载。

### Ubuntu 多用户管理

- **多人共用服务器，最好给每个人单独分配一个账户**，避免环境冲突，数据被误删，也可以监视各个用户的资源占用。
- **尽可能保证 root 和 sudo不直接用于运行自己的程序**。 用户只用于维护服务器，和安装一些所有用户共用的程序 比如 anaconda， matlab，VIM， cuda。
- 使用sudo用户安装 共用程序到 `/usr/local` 中， 然后个人用户在自己用户环境中添加这些程序到*用户系统环境中*。  ` vim ~/.bashrc`.  目前我们的服务器只在公用账户里安装了“anaconda， matlab,  Vim,  cuda”
- 添加新用户命令

```
useradd newuser   # ubuntu 添加新用户， 用户名为：newuser
passwd newuser    # 为newuser  指定密码
grep bash /etc/passwd  # 查看现有的所有用户
```

### ubuntu 远程控制

服务器一般噪音都比较大，所以一般都是放在机房，然后用户用远程进行连接操作。

- 远程软件：
  - [Xshell (用的人比较多)]([XSHELL - NetSarang Website](https://www.xshell.com/zh/xshell/))，所有任务都用命令行实现。
  - [Bitvise SSH Client(强烈推荐)]([Download Area | Bitvise](https://www.bitvise.com/download-area)). 支持可视化SFTP文件窗口，支持文件窗口拖拽，对windows用户十分友好，可以多开终端窗口，一个窗口运行程序，一个监测资源十分好用。

### Ubuntu 常用资源监测命令

多人共用的时候，很容出现资源不足的问题。 有可能是有人在运行程序占用，也有可能是僵尸进程把资源占用了，后一种情况在用tensorflow 开启多进程时，十分常见。 这就需要查看系统资源占用，查看哪个用户，哪些程序占用的，是否为僵尸进程（这个我一般是联系该用户询问的）。

```
free      # 查看当前内存使用情况（单位为kB）
ps –a     # 查看当前用户的所有进程
ps aux    # 查看所有进程
top       # 动态显示所有进程， 
          # 按下‘c’：可以查看具体的程序名； 
          # 按下‘t’：可以查看当前CPU占用
          # ctrl+z: 推出动态监测 
ps auxw | head -1;ps auxw|sort -rn -k4|head -5 #  显示内存占用前5
ps auxw|head -1;ps auxw|sort -rn -k3|head -3   #  显示cpu资源占用前3

nvidia-smi  # 查看当前GPU的使用状况，cuda监测命令
nvidia-smi –q –d memory # 查看gpu内存
nvidia-smi –q  # 查看当前gpu信息
nvidia-smi –L  # 显示当前系统下gpu的个数
```

###  ubuntu 基础命令

```
cd /path     # 切换目录到 /path
ls           # 罗列当前目录下的所有文件和文件夹

Mkdir newdir # 新建文件夹 文件名为newdir
rm –f file   # 强制删除文件
rmdir -rf    # 强制删除目录
mv /A/file /B  # 将文件从一个文件夹A移动到文件夹B
copy A B       # 复制文件夹A到B
```



## python 环境

- 安装 [anaconda](https://www.anaconda.com/)， 里面几乎涵盖了python的基础包，十分方便，自带pip包管理。

- 强烈建议使用**conda 虚拟环境来管理个人的python环境**， python的强大在于他有各种各样的第三方包，可以避免重复造轮子。但成也萧何，败也萧何， python版本和第三方包版本的依赖有严格的对应关系，我们可能同时需要多个互不干扰的python版本。尤其在复现别人用torch，tensorflow等框架写的code的时候，最好每个项目建立一个虚拟环境，让他们相互不干扰，避免辛辛苦苦为新项目配好了一个环境，以前的项目却跑不了的尴尬。

  ```
  conda create -n Newenv python=x.x # 创建名为Newenv的虚拟环境， x.x 为python版本
  conda env list	# 罗列所有虚拟环境,主要用于找到环境名，多了容易记不住
  source activate env_name	# linux 下激活虚拟环境指令
  ...                         # 在这个环境中安装所需的包，运行程序
  
  source deactivate	# linux 推出当前虚拟环境（window环境前面不需要source）
  
  conda remove -n env_name --all	# 删除虚拟环境
  
  conda list  # 查看安装了哪些包
  conda updata conda	# 检查更新当前conda
  # 查看当前源 
  conda config --show-sources
  ```

  

  

  

  pip 和 conda 管理python的第三方包





# 分界线

----





## 程序进程重命名：便于管理服务器上的程序。

```python
import setproctitle
setproctitle.setproctitle('yongjie_du')
```

## 管理命令

1. 查看所有用户：

```

```



## matlab 

```
%% 不启动桌面程序，不显示版本号，运行
$ matlab -nodesktop -nosplash -r matlabfile
```



## python 项目依赖环输出 requirements.txt

- 为python项目生成一个 `requirements.txt` 文件

```

有两种方法：
- 1， pip freeze > path/requirements.txt   # 这种方法会打包输出环境中所有的包

- 2， pipreqs rootpath_project  # rootpath_project: 项目的根目录
      pipreqs rootpath_project  --encoding=utf-8 --force # 覆盖目录下已经有的requirements.txt
      
      pip install pipreqs  # 安装包
```

- 安装 `requirements.txt` 中的包 

  `pip install -r requirements.txt`

## Ubuntu 文件权限管理

```
# 查看文件权限
ls -ld foldname  # 查看文件夹 权限
ls -l filename   # 查看文件权限

## 修改 文件/目录 拥有者
chown -R nali foldname # 修改foldname 的所有着是nali

#chown [who] [+|-|=] [mode] name
#who: u,表示“用户（user）”，即文件或目录的所有者
#	 g: 表示“同组（group）用户”，即与文件属主有相同组ID的所有用户。
#	 o: 表示“其他（others）用户”。
#	 a: 表示“所有（all）用户”。它是系统默认值。
#	 操作符号可以是：
#     + 添加某个权限。
#     - 取消某个权限。
#     = 赋予给定权限并取消其他所有权限（如果有的话）。
#mode：
#	 r 可读。
#	 w 可写。
#	 x 可执行。
	 
## chgrp 改变文件或目录所属的组
chgrp -R book opt/local 改变/opt/local/book 及其子目录所属的组为book
	 
```





## 判断平台

```python
import platform
 
if __name__ == '__main__':
    sys = platform.system()
    if sys == "Windows":
        print("OS is Windows!!!")
    elif sys == "Linux":
        print("OS is Linux!!!")

```



## 1. 程序运行

```linux command
nuhup python -u test.py > test.log 2>&1 &
```

\1. 最后一个“&”表示后台运行程序

\2. “nohup” 表示程序不被挂起

\3. “python”表示执行python代码

\4. “-u”表示不启用缓存，实时输出打印信息到日志文件（如果不加-u，则会导致日志文件不会实时刷新代码中的print函数的信息）

\5. “test.py”表示python的源代码文件

\6. “test.log”表示输出的日志文件

\7. “>”表示将打印信息重定向到日志文件

\8. “2>&1”表示将标准错误输出转变化标准输出，可以将错误信息也输出到日志文件中（0-> stdin, 1->stdout, 2->stderr）

————————————————

原文链接：https://blog.csdn.net/b876144622/article/details/81047967

##  rz 上传

## sz 下载

## 查看进程

## 文件操作








## 5.  程序安装（pip包）

`Pip install --user namepackage` <!--：将包安装在当前用户目录下-->

`pip install programname`

`unzip filename.zip`: <!--解压-->

`pip uninstall packagename`<!--: 卸载已经安装的函数包-->

`pip show` ：<!--查看所有已经安装的包-->

`pip install --upgrade packagename`<!--:升级包-->

`pip list --outdated`<!--:查看需要更新的包-->

## 6.  Arch linux程序安装：

`pacman -S gedit`  安装gedit文本编辑工具，远程不容易操作（基本没用）

## 7.  linux 资源查看命令：



## 8.  显示当前文件夹路径：

`pwd` 

## 9.  添加用户 



```

```



## 10. 卸载

`yum remove appfile`

 

## 11. 安装tensorflow-gpu cuda keras 出现的问题

 

## 12. pip

### pip 报错 -bash

安装pip后，终端输入pip 报错-bash，然而切换到pip的安装目录、/usr/local/python3/bin 运行：python pip却可以正常输出，

解决办法：更改配置文件：`vi ~/.bash_profile`

将“export PATH ”加上python的bin路径

变成：“export PATH = /usr/local/python3/bin:$PATH”保存，退出vi

在终端输入source ~/.bash_profile 使更改的配置文件生效

\2. pip 安装其他包的时候，出现tsl/ssl not available 证明安装python时没有加载ssl模块

解决办法：重新编译安装python：cd pythonx.x 

 运行：./configure –with-ssl

​     `make`  

​     `make install`

 

###  pip国内源

`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package`     临时使用

```
`阿里云 http://mirrors.aliyun.com/pypi/simple/
 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
 豆瓣 http://pypi.douban.com/simple
 Python官方 https://pypi.python.org/simple/
 v2ex http://pypi.v2ex.com/simple/
 中国科学院 http://pypi.mirrors.opencas.cn/simple/
 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/`
```

 永久更改pip

1. mkdir ~/.pip

2. vim ~/.pip/pip.conf

   修改其内容为：

   ```
   [global]
   index-url = https://pypi.tuna.tsinghua.edu.cn/simple
   [install]
   trusted-host = https://pypi.tuna.tsinghua.edu.cn
   ```

   

## 13. 显卡操作

`nvidia–smi` :查看当前gpu的显存占用量



 

## 14. cuda版本

`cat /usr/local/cuda/version.txt` :查看cuda版本

`cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2` ：查看cudnn版本

`nvcc –version` :查看cuda版本（验证cuda是否安装成功）

## 15. 更新nvidia驱动

需要先卸载已有的nvidia驱动，`yum remove nvidia*`

## 16. 文件解压

`unzip filename.zip`：解压到当前文件夹

`zip –r filename.zip ./*`: 压缩当前目录下的全部文件为filename.zip

## 17. 查看系统用户：

`cat /etc/group`

## 18. 文本编辑命令：

`Vim *.py`

`Esc` 退出插入模式

`Shift+`：  进入命令输入模式，输入wq：保存退出，q只退出不保存

## 19. Python程序运行

`Python pythontest.py >pythontest.log &` :将系统输出输入到pythontest.log中，并使程序在后台运行（&）

`nohup Python -u pythontest.py >outfilename.log 2>&1 &`  将程序的输出，写入到outfilename.log中，并且无缓存，直接写入。

 

## 20. 程序别名

https://www.cnblogs.com/i0ject/p/3658084.html

`alias matlab=/usr/local/Matlab/R2018b/bin/matlab`

 

## 21. 挂载iso文件

`sudo mkdir /media/tmp :创建加载文件夹`

`sudo mount –o loop [path]/filename.iso /media/tmp ：加载文件`

`sudo umount media/tmp`



## 22. python在ubuntu系统和window系统对文件操作的不同

`windows： 所有的字符 / 或者 \， 不管是一个或者多个，都会统一转换成一个 \` 

` ubuntu:  字符 `/`, 不管是一个或者多个，都统一转换成一个 `/`, 对于字符`\`, ubuntu 不认为这是一个合法的文件路径字符，如果存在，不会进行处理。 `

结论：为了方便兼容，在python有文件路径的，一律使用`/`作为分隔符





## 23. pytorch

ryzen服务器上：

在conda 虚拟环境中安装

激活： `source activate pytorch`

关闭：`source deactivate`

pip：安装包的时候如果出现，下载中途报错（一片红），直接复制package地址，在本地下载好，上传到服务器，手动安装： `pip install pakagename.whl`



## 24. conda

 创建python虚拟环境，

`conda create -n env_name python=x.x # env_name 文件可以在anaconda/env 下找到`

conda 常用命令：

```linux


```



### tesla-V100 8GPU 两个conda环境的配置办法

```
/usr/local/anaconda3  公用
/home/amax/anaconda3  私有


source activate /user/local/anaconda3  # 激活公用anaconda

## 迁移conda 环境 

# 1 clone || 不要轻易用，不太了解机制|| 本地复制一个 新的虚拟环境。 
conda create --name newEnv_name --clone oldEnv_name  # 创建本地备份或快照 newEnv_name:新环境， newEnv_name:旧环境。

# 迁移 environment.yml
conda env export > environment.yml   # 导出环境的依赖文件 || 虚拟环境共享
conda env create -f environment.yml	 # 根据依赖文件重新安装环境

# 迁移 requirement.txt
pip freeze > requirements.txt      # 生成依赖包文件
conda create -n envname python=3.7 # 新建虚拟环境
pip install -r requirements.txt    # 安装依赖包

###################### 推荐 #################
## 以上三种迁移方法都是，保存列表，然后在目标机上联网重新下载安装，
## 下面介绍一种，不联网的方法，但是需要目标机和源机anaconda 版本相似
mkdir -p /home/yongjiedu/.conda/envs
sudo cp /home/amax/.conda/envs/* /home/yongjiedu/.conda/envs #   直接复制envs文件夹到目标机
conda env list  # 检查envs环境
```

### conda 换源

1. `conda config --show-source`

2. 更换 .condarc  为以下内容

   ```
   show_channel_urls: true
   channels:
     - http://mirrors.bfsu.edu.cn/anaconda/pkgs/main
     - http://mirrors.bfsu.edu.cn/anaconda/pkgs/r
     - http://mirrors.bfsu.edu.cn/anaconda/pkgs/msys2
     - defaults
   custom_channels:
     conda-forge: http://mirrors.bfsu.edu.cn/anaconda/cloud
     msys2: http://mirrors.bfsu.edu.cn/anaconda/cloud
     bioconda: http://mirrors.bfsu.edu.cn/anaconda/cloud
     menpo: http://mirrors.bfsu.edu.cn/anaconda/cloud
     pytorch: http://mirrors.bfsu.edu.cn/anaconda/cloud
     simpleitk: http://mirrors.bfsu.edu.cn/anaconda/cloud
   ```

   3. `conda clean -i   # 清空缓存的源`