# Python 环境安装

## Python 3.7.9 安装
1. 安装各种依赖包
```
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel
```

2. 下载并安装 3.7.9
```
wget "https://www.python.org/ftp/python/3.7.9/Python-3.7.9.tgz"
tar -zxvf Python-3.7.9.tgz
```

3. 编译安装
```
cd Python-3.7.9
./configure --prefix=/usr/local/python3
make && make install
```

4. 创建软链接
```
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

## pyenv 安装

## pyenv 安装 Python 版本加速
将 Python 包放到 ~/.pyenv/cache/ 目录下
然后执行 pyenv install $version

## virtaulenv 安装使用
在线安装方式
```
pip install virtaulenv
```

离线安装方式
```
yum search virtualenv
yum install python3-virtualenv
```

virtualenv -p ~/.pyenv/versions/3.7.9/bin/python venv


## 离线安装第三方包（迁移虚拟环境）
### 方法一（不同架构）
在在线环境
```
source venv/bin/active
pip install pipreqs
pip install pip-download
pipreqs ./ --force
pip-download -r requirements.txt -d pip_download_arm64 -py 37 -p manylinux1_arm64
```
在离线环境
```
source venv/bin/active
pip install --no-index --find-links=pip_download_arm64 -r requirements.txt
```

### 方法二（相同架构）
```
拷贝 venv
修改参数路径
```
