---
layout: editorial
---

# pip

### pip安装

`pip` 是 `Python` 包管理工具，该工具提供了对Python 包的查找、下载、安装、卸载的功能。`Python 2.7.9 +` 或 `Python 3.4+` 以上版本都自带 `pip` 工具。

若未安装，使用一下命令进行安装

```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py 
sudo python get-pip.py
```

### pip常用命令

#### 安装与卸载包

以numpy为例

```bash
# 安装
pip install numpy

# 安装指定版本
pip install numpy==1.12.0

# 指定安装最小版本
pip install numpy>=1.12.0

# 卸载
pip uninstall numpy
```

#### 展示安装信息

```bash
pip show 

#查看指定包的详细信息
pip show -f numpy

#显示所有包
pip list
```

#### 批量安装与导出

```bash
# 导出当前环境的包至requirement.txt
pip freeze > requirement.txt

# 安装requirement.txt中所有包
pip install -r requirement.txt
```

#### 其他

```bash
# pip升级
pip install --upgrade pip

#查看pip版本
pip -v	
```

### pip换源

在 `python` 里经常要安装各种这样的包，安装各种包时最常用的就是 `pip`，`pip` 默认从官网下载文件，官网位于国外，下载速度时快时慢，还经常断线，国内的体验并不太好。

解决办法是把 `pip` 源换成国内的，最常用的并且可信赖的源包括清华大学源、豆瓣源、阿里源。

#### 临时换源

以清华大学源为例

```bash
pip install markdown -i https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 永久换源

```bash
# 清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# 或 阿里源
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/

# 或 腾讯源
pip config set global.index-url http://mirrors.cloud.tencent.com/pypi/simple

# 或 豆瓣源
pip config set global.index-url http://pypi.douban.com/simple/
```
