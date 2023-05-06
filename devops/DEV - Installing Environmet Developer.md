# Installing Environment Developer



<details><summary>1. Instalando compilador e outras ferramentas</summary><p>

```sh
# atualizando repositorio
sudo apt update

# instalando ferramentas
sudo apt install -y build-essential g++-10 cmake git \
  apt-transport-https ca-certificates curl gnupg lsb-release \
  autoconf automake libtool curl make unzip xz-utils \
  libboost-all-dev libspdlog-dev  libhiredis-dev  libssl-dev  libfmt-dev

# alterando o compilador default
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-10 10
  
```


</p></details>
<details><summary>2. Instalando o Boost Manualmente</summary><p>

```sh
mkdir -p ~/lib-external
pushd ~/lib-external

# remocao do projeto atualmente instalado
sudo apt autoremove -y libboost-all-dev

# download
wget https://boostorg.jfrog.io/artifactory/main/release/1.74.0/source/boost_1_74_0.tar.bz2   --no-check-certificate
tar -jxvf boost_1_74_0.tar.bz2

# compilacao
cd boost_1_74_0/
./bootstrap.sh
./b2

# instalacao
sudo ./b2 install

popd
```


</p></details>
<details><summary>3. Preparando bibliotera do Redis CPP</summary><p>


```sh
# instalar prequisitos
sudo apt install  -y  libhiredis-dev

mkdir -p ~/lib-external
pushd ~/lib-external

# export GIT_SSL_NO_VERIFY=1
git config --global http.sslverify false
git clone https://github.com/sewenew/redis-plus-plus.git
cd redis-plus-plus

# preparando o projeto para compilacao
mkdir build
cd build
sudo apt install cmake
cmake -DREDIS_PLUS_PLUS_CXX_STANDARD=20 ..

# compilando o projeto
make -j3
sudo make install

## realizando o teste com o Redis
./test/test_redis++ -h 127.0.0.1 -p 6379
popd
```


</p></details>
<details><summary>4. Instalando a Biblioteca ZLIB</summary><p>


```bash
mkdir -p ~/lib-external
pushd ~/lib-external

wget https://github.com/madler/zlib/archive/v1.2.12.tar.gz
tar -xvf v1.2.12.tar.gz
rm -fr v1.2.12.tar.gz

cd zlib-1.2.12/
./configure ; make
sudo make install

```


</p></details>
<details><summary>5. Instalando o Google ProtoBuf</summary><p>


```bash
sudo apt-get install  -y  autoconf automake libtool curl make g++ unzip


mkdir -p ~/lib-external
pushd ~/lib-external

rm -fr protobuf
# export GIT_SSL_NO_VERIFY=1
git config --global http.sslverify false
git clone https://github.com/protocolbuffers/protobuf.git
cd protobuf
# git checkout v3.19.1
git submodule update --init --recursive
./autogen.sh

./configure CXXFLAGS="-Dprotobuf_WITH_ZLIB=ON -Dprotobuf_BUILD_TESTS=OFF -DBUILD_EXAMPLES=OFF" ; make -j6 ; make check

sudo make install

# refresh shared library cache.
sudo ldconfig 
popd
```


</p></details>
<details><summary>6. Compilacao e Instalacao do QuickFix</summary><p>


- Instruções:

```sh
mkdir -p ~/lib-external
pushd ~/lib-external

#  wget https://github.com/quickfix/quickfix/archive/v1.15.1.tar.gz -O quickfix-1.15.1.tar.gz
# tar -xvf quickfix-1.15.1.tar.gz
# rm -fr quickfix-1.15.1.tar.gz
# cd quickfix-1.15.1/

rm -fr quickfix-master
git clone https://github.com/quickfix/quickfix.git  quickfix-master
cd quickfix-master

# necessario para os scripts do "CHECK"
sudo apt  install ruby -y

./bootstrap
./configure CXXFLAGS="-DBUILD_SHARED_LIBS=OFF"
make -j8
# make check -j8

sudo make install
```
