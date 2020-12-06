安装依赖包
yum install -y gcc gcc-c++ make automake openssl-devel

下载Cmake
wget https://cmake.org/files/v3.6/cmake-3.6.2.tar.gz

解压Cmake
tar xvf cmake-3.6.2.tar.gz && cd cmake-3.6.2/

编译安装cmake
./bootstrap
gmake
gmake install

查看编译后的cmake版本
/usr/local/bin/cmake --version

移除原来的cmake版本
yum remove cmake -y

新建软连接
ln -s /usr/local/bin/cmake /usr/bin/

终端查看版本
cmake --version