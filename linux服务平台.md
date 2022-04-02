# linux 服务平台

## 写在前面的话

- 主要总结自己几年来，维护办公室用于机器学习的服务器的一些心得。办公室服务器用的是Ubuntu16.4LTS.

- 内容顺序根据使用顺序排列.



## 目录

[toc]

 

   

## 系统管理

### 安装Ubuntu

- 使用U盘安装，网上很多教程，目前用Ubuntu16.4,LTS  和18.4LTS的居多
- 目前服务器都是固态（小储量）+机械硬盘（大储量），**安装系统到固态硬盘上，一定要把机械硬盘挂载在` /home` 文件目录上**，不然默认的是在固态硬盘上，这出现硬盘的存储空间不足的问题。
- 忘记把机械硬盘挂载在` /home` 也可以手动迁移/home  上的文件到机械硬盘，然后再完成挂载。

### Ubuntu 多用户管理

- **多人共用服务器，最好给每个人单独分配一个账户**，避免环境冲突，数据被误删，也可以监视各个用户的资源占用。
- **尽可能保证 root 和 sudo不直接用于运行自己的程序**。 用户只用于维护服务器，和安装一些所有用户共用的程序 比如 anaconda， matlab，VIM， cuda。
- 使用sudo用户安装 共用程序到 `/usr/local` 中， 然后个人用户在自己用户环境中添加这些程序到*用户系统环境中*。  ` vim ~/.bashrc`.  目前我们的服务器只在公用账户里安装了“anaconda， matlab,  Vim,  cuda”
- 添加新用户命令

```
useradd newuser   # Ubuntu 添加新用户， 用户名为：newuser
passwd newuser    # 为newuser  指定密码
grep bash /etc/passwd  # 查看现有的所有用户
cat /etc/group
```

### Ubuntu 远程控制

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

###  Ubuntu 基础命令

```
cd /path     # 切换目录到 /path
ls           # 罗列当前目录下的所有文件和文件夹
pwd          # 显示当前文件夹路径

Mkdir newdir # 新建文件夹 文件名为newdir
rm –f file   # 强制删除文件
rmdir -rf    # 强制删除目录
mv /A/file /B  # 将文件从一个文件夹A移动到文件夹B
copy A B       # 复制文件夹A到B

unzip filename.zip # 解压到当前文件夹
zip –r filename.zip ./*` # 压缩当前目录下的全部文件为filename.zip
```



### 使用 Vim 编辑文本

vim是大佬们做文本编辑的利器, 可以做到全部操作都在键盘上操作, 以及一些自动化批处理操作.  **然而对于一般人这属于锦上添花**, 只需要掌握简单的指令就可以了. 

- vim有两个界面,命令界面(默认), 字符插入界面,(这个界面可以直接想我们修改txt文件一样)

  ```
  vim  test.txt # 新建或者编辑当前目录下的test.txt, 
  i    # 进入字符插入界面, 命令窗左下角会有'insert'
  	 # 移动光标编辑内容
  esc  # 推出字符修改界面, 命令窗左下角会'insert' 会消失
  :q    # 退出编辑, 在命令界面才生效
  :q!   # 强制退出编辑
  :wq   # 保存并退出
  ```

  

### 跑自己的程序

- python 后台执行程序     [ref](https://blog.csdn.net/b876144622/article/details/81047967)

  ```
  nuhup python -u test.py > test.log 2>&1 &
  
  # 最后一个“&”表示后台运行程序
  # “nohup” 表示程序不被挂起
  # “python”表示执行python代码
  # “-u”表示不启用缓存，实时输出打印信息到日志文件（如果不加-u，则会导致日志文件不会实时刷新代码中的print函数的信息）
  # “test.py”表示python的源代码文件
  # “test.log”表示输出的日志文件
  # “>”表示将打印信息重定向到日志文件
  # “2>&1”表示将标准错误输出转变化标准输出，可以将错误信息也输出到日志文件中（0-> stdin, 1->stdout, 2->stderr）
  
  Python pythontest.py >pythontest.log & :将系统输出输入到pythontest.log中，并使程序在后台运行
  ```

  

- matlab 运行程序不启动图形界面

  ```
  matlab -nodesktop -nosplash -r test  # 运行test.m 程序
  
  # -nosplash: 不显示启动画面(版权页)
  # -nodesktop: 不启动桌面环境
  ```

  ​    

## python 环境

- 安装 [anaconda](https://www.anaconda.com/)， 里面几乎涵盖了python的基础包，十分方便，自带pip包管理。

- **更改conda源，pip源为国内镜像**， 由于cionda和pip的官方源（所谓源就是默认的下载地址）都是在国外，由于墙的存在，正常访问这些源很慢，甚至无法下载，所以要把这些源更换为国内的镜像网站。这里我们使用清华源为例，还有阿里源，中科院的源，淘宝源等等。

  [更改pip源](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)

  ```
  pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package # 临时使用清华源下载
  ## 永久更改（建议使用）
  mkdir ~/.pip   # 新建.pip 文件夹，如果已存在，跳过这一步
  vim ~/.pip/pip.conf  # 新建并编辑pip.conf 文件 文件内容如下
  ```

  <center> ~/.pip/pip.conf</center>
  ```
  [global]
  index-url = https://pypi.tuna.tsinghua.edu.cn/simple
  [install]
  trusted-host = https://pypi.tuna.tsinghua.edu.cn
  ```
  
  <center>常用的pip源镜像</center>
  ```
   阿里云 http://mirrors.aliyun.com/pypi/simple/
   中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
   豆瓣 http://pypi.douban.com/simple
   Python官方 https://pypi.python.org/simple/
   v2ex http://pypi.v2ex.com/simple/
   中国科学院 http://pypi.mirrors.opencas.cn/simple/
   清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
  ```
  
  
  
  [更改conda源](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)
  
  ​	
  
  ```
    conda config --show-source  # 查看当前系统默认的源
    
    # 永久更换默认源
    vim ~/.condarc    # 打开编辑conda的配置文件, ~:表示当前用户的主目录
                      # 把下面的内容 复制到 ~/.condarc，保存
    conda clean -i    # 清空缓存的源`
    source ~/.condarc # 使配置文件生效
  ```
  
  <center>更改源后 ~/.condarc  中的内容</center>
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
  
  


- 强烈建议使用**conda 虚拟环境来管理个人的python环境**， python的强大在于他有各种各样的第三方包，可以避免重复造轮子。但成也萧何，败也萧何， python版本和第三方包版本的依赖有严格的对应关系，我们可能同时需要多个互不干扰的python版本。尤其在复现别人用torch，tensorflow等框架写的code的时候，最好每个项目建立一个虚拟环境，让他们相互不干扰，避免辛辛苦苦为新项目配好了一个环境，以前的项目却跑不了的尴尬。

    ```
    conda create -n Newenv python=x.x # 创建名为Newenv的虚拟环境， x.x 为python版本
    conda env list	# 罗列所有虚拟环境,主要用于找到环境名，多了容易记不住
    source activate env_name	# linux 下激活虚拟环境指令
                                # 在这个环境中安装所需的包，运行程序
    source deactivate	# linux 推出当前虚拟环境（window环境前面不需要source）
    conda remove -n env_name --all	# 删除虚拟环境
    ```



- pip 和conda **在线**安装，卸载第三方包。注意使用pip和conda都可以安装第三方包，方便程度差不多，但是由于源的问题，有时候有的包只能用pip装，也有的只能用conda安装，大家灵活使用。

  ```
  pip install packagename [--user] # 安装名为packagename  [--user] 为可选参数，表示安装在当前用户目录
  pip uninstall packagename        # 卸载
  pip list   # 罗列所有已经安装的包
  pip show  package # 显示已经按照的packaege的详细信息：版本，位置，等等
  ```

  对应的conda的命令

  ```
  conda install packagename
  conda uninstall packagename
  conda list
  ```

  **注意有时候，由于一些包并没有在镜像源里面，所以在线安装会出现访问超时**，**这时候我们可以使用本地安装**windows浏览器下载对应的包（.whl文件）.上传到linux,

  ```
  # 这里以/home/myname/download/a.whl 为例
  cd /home/myname/download  # 切换到对应目录
  pip install  a.whl   # 执行本地安装
  ```

  

  

  


## Matlab 环境

  - [ ] matlab 的安装涉及到挂载iso文件
  - [ ] 安装相对麻烦
  - [ ] 把共用账户里面安装的matlab 添加到其他用户的环境变量里





## 一些典型应用场景

### 跨平台运行场景

 一些参数在不同环境设置不同。 我们写在windows上写程序，在linux上运行，但是有些路径不一致，这时候就需要判断平台，给不同的平台不同的参数

```
##  python 区分linux和windows平台
import platform
sys = platform.system()
if sys == "Windows":
    print("OS is Windows!!!")
elif sys == "Linux":
    print("OS is Linux!!!")
```

###  Ubuntu 挂载iso文件

```
sudo mkdir /media/tmp  # 创建挂载点（文件夹）
sudo mount –o loop [path]/filename.iso /media/tmp ： # 把iso文件挂载到挂载点
                       # 此时iso中的文件就被映射到  /media/tmp 文件夹里面了
sudo umount media/tmp  # 解除挂载
```



### Ubuntu 文件权限管理

有时需要把文件从一个系统用户那里迁移到另一个用户哪里，可以在sudo用户中使用文件移动或者复制的方法移动，此时目标用户只能读取这些文件，无法修改（因为没有修改权限）。 这时候就要修改文件权限。



- linux中的文件/目录对于其潜在的访问者，有着严格的权限限制。

- 文件/目录 对应的实体（潜在的访问者）被分为三类： 
  - 文件的拥有者（系统的某个用户）
  - 拥有者在的组（group）
  - 其他组
- 文件/目录 的三个权限（每个实体都对应三个权限）
  - r  ： 读取
  - w ： 修改
  - x ：  执行

```
# 查看文件权限
ls -ld foldname  # 查看文件夹 权限
ls -l filename   # 查看文件权限

#    输出解析
#  yongjiedu@amax2021:~$ ls -ld test.txt
#  -rw-rw-r-- 1 yjiedu yjiedu 59 4月   2 16:08 test.txt
#  1234567890    user    group   字节数
# 1[-]:表示为文件，d表示目录
# 234[rw-]: 拥有者的权限， 读写
# 567[rw-]: 文件所在组的权限， 读写
# 890[r--]: 其他组的权限， 只读



## 修改 文件/目录 拥有者 [CHange OWN]
chown -R yjiedu foldname # 修改foldname 的所有者是yjiedu

## chgrp 改变文件或目录所属的组 [CHange GRouP]
chgrp -R book opt/local 改变/opt/local/book 及其子目录所属的组为book

## 修改文件/目录 权限
#chmod [who] [+|-|=] [mode] name
#who: u: 表示“用户（user）”，即文件或目录的所有者
#	  g: 表示“同组（group）用户”，即与文件属主有相同组ID的所有用户。
#	  o: 表示“其他（others）用户”。
#	  a: 表示“所有（all）用户”。它是系统默认值。
#	 操作符号可以是：
#     + 添加某个权限。
#     - 取消某个权限。
#     = 赋予给定权限并取消其他所有权限（如果有的话）。
#mode：
#	 r 可读。4
#	 w 可写。2
#	 x 可执行。1
	 
# e.g.
chomd u=rwx,go=rx text.txt # 设置text.txt的拥有者的权限为rwx，所在组和其他组的权限为rx
chmod a=rx,g-x,o-x text.txt # 设置text.txt的拥有者的权限为rwx，所在组去除执行权限，其他组去除执行权限
chmod a=rx,go+w text.txt    # 设置text.txt的拥有者的权限为rwx，所在组和其他组增加写的权限
```



### python程序进程重命名

便于管理服务器上的程序，可以把python程序对应的进程名字自定义改变。

```python
import setproctitle
setproctitle.setproctitle('yjiedu')
```



### 输出python 输出requirements.txt

输出依赖环境到 requirements.txt

```
## 为python项目生成一个 requirements.txt 文件
# 方法1
pip freeze > path/requirements.txt   # 这种方法会打包输出环境中所有的包
# 方法2
pip install pipreqs  # 安装包
pipreqs rootpath_project  # rootpath_project: 项目的根目录
pipreqs rootpath_project  --encoding=utf-8 --force # 覆盖目录下已经有的requirements.txt

## 安装 `requirements.txt` 中的包 
pip install -r requirements.txt
```



### python不同平台的文件分隔符

- windows： 所有的字符【 / 】或者【 \】， 不管是一个或者多个，统统视为1个【`/`】 

- Ubuntu:  是一个或者多个【字符 `/`】，都统一转换成一个【 `/`】, 字符【`\`】 在linux的python中不是合法文件分隔符

- 结论：为了方便兼容，在python有文件路径的，一律使用`/`作为分隔符



### 迁移conda虚拟环境

```
/usr/local/anaconda3  公用
/home/yjiedu/anaconda3  私有

source activate /user/local/anaconda3  # 激活公用anaconda

## 迁移conda 环境 

# clone  本地复制一个新的虚拟环境。|| 收集旧环境里面的包，在新的环境中重新在线安装 
conda create --name newEnv_name --clone oldEnv_name  # 创建本地备份或快照 newEnv_name:新环境， newEnv_name:旧环境。

# 迁移 environment.yml  || 在线重新安装
conda env export > environment.yml   # 导出环境的依赖文件 || 虚拟环境共享
conda env create -f environment.yml	 # 根据依赖文件重新安装环境

# 迁移 requirement.txt  || 在线重新安装
pip freeze > requirements.txt      # 生成依赖包文件
conda create -n envname python=3.7 # 新建虚拟环境
pip install -r requirements.txt    # 安装依赖包

###################### 推荐 #################
## 以上三种迁移方法都是，保存列表，然后在目标机上联网重新下载安装，
## 下面介绍一种，不联网的方法，但是需要目标机和源机anaconda 版本相似
mkdir -p /home/huali/.conda/envs
sudo cp /home/yjiedu/.conda/envs/* /home/huali/.conda/envs #   直接复制envs文件夹到目标机
conda env list  # 检查envs环境
```

 

###  Ubuntu 命令重定义

-  linux提供了一个命令重映射命令， alias 用于自定义映射长命令为短字符，我们可以自定义一个aliases文件存放我们的重定义命令。

```
vim ~/.bash_aliases   # 新建或者编辑 alias存放文件
alias matlab=/usr/local/Matlab/R2018b/bin/matlab  # 把这条命令添加到文件最后，保存推出

source ~/.bash_aliases  # 使配置文件生效
```
