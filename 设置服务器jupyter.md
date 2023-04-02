## jupyter notebook 服务端配置



- 安装 `conda install jupyter notebook`
- 生成一个密码

```
## 利用python设置一个密码,生成这个密码的密钥写入jupyter的设置里面，用于登录jupyter
from notebook.auth import passwd
passwd()
## 这时候会让你输入密码，确认密码，然后输出一个密钥，我们输入123，并确认，然后复制这个密钥如下：
argon2:$argon2id$v=19$m=10240,t=10,p=8$s2fzheBaIDgGp4/n5uUziQ$dzFhe6wN+bMr7lRiBGAmMot5Xe/cDVJWFBGSPO8X4W4

```

- 设置服务端的jupyter

  ```
  ## 生成jupyter的设置文件
  jupyter notebook --generate-config
  ## 编辑 设置文件
  vim ~/.jupyter/jupyter_notebook_config.py
  ```

  主要修改以下几项，下面几项都是注释状态的，把注释去掉，并且做出对应修改，我的修改如下

  ```
  # ip 地址
  c.NotebookApp.ip='192.168.1.112' # 这里一定要填服务器的IP
  # 添加密钥
  c.NotebookApp.password=u'argon2:$argon2id$v=19$m=10240,t=10,p=8$s2fzheBaIDgGp4/n5uUziQ$dzFhe6wN+bMr7lRiBGAmMot5Xe/cDVJWFBGSPO8X4W4'
  # 设置自动启动浏览器
  c.NotebookApp.open_browser=False
  # 设置端口
  c.NotebookApp.port=8889
  ```

  然后就可以在本地浏览器中访问 `192.168.1.112:8889`，输入设置的密码123，就可以使用了





**参考**

---

[1][](https://zhuanlan.zhihu.com/p/431003426)