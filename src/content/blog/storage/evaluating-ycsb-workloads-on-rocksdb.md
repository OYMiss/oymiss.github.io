---
author: Chongzhuo Yang
title: "Evaluating YCSB Workloads on RocksDB: Build and Execution"
pubDatetime: 2024-06-19 20:20:25
tags:
  - storage
slug: "evaluating-ycsb-workloads-on-rocksdb"
# draft: true
description: An experiment log for recording steps to set up the environment for testing rocksdb on ycsb.
---

## Server Setup

### Connect Server

Add generated public keys to server.

```bash
ssh-keygen
vim ~/.ssh/authorized_keys
chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys
```

#### Method 1: SSH command

```bash
ssh -J USTC-port user-real@10.0.0.1
```

#### Method 2: SSH config

```bash
Host node1
  HostName 10.0.0.1
  ProxyJump USTC-port
  User user-real
  IdentityFile ~/.ssh/id_ed25519
  Port 22

Host USTC-port
  HostName 1.2.3.4
  User user-jump
  Port 4321
```

Then, we can access `node1` using `ssh node1`.

### Update GCC

> We should check GCC and G++ firstly!

Download and build GCC from source.

```bash
GCC_VERSION=9.2.0
# https://mirrors.ustc.edu.cn/gnu/gcc/
wget https://ftp.gnu.org/gnu/gcc/gcc-${GCC_VERSION}/gcc-${GCC_VERSION}.tar.gz
tar xzvf gcc-${GCC_VERSION}.tar.gz
mkdir obj.gcc-${GCC_VERSION}
cd gcc-${GCC_VERSION}
# maybe slow due to network
./contrib/download_prerequisites
cd ../obj.gcc-${GCC_VERSION}
# install to home
../gcc-${GCC_VERSION}/configure --prefix=$HOME/gcc-${GCC_VERSION} --disable-multilib --enable-languages=c,c++
make -j 8
make install
```

Add new path to `.bashrc`.

```bash
GCC_PATH=/home/xxx/gcc-9.2.0
export PATH=$GCC_PATH/bin:$PATH
export LD_LIBRARY_PATH=$GCC_PATH/lib64:$LD_LIBRARY_PATH
```

## Expriments

### Build RocksDB

```bash
# clone rocksdb
git clone -b 6.4.tikv https://github.com/tikv/rocksdb.git
cd rocksdb
make shared_lib -j 8
make install-shared INSTALL_PATH=/home/chongzhuo/usr/
```

### Local Install

A useful [link](https://unix.stackexchange.com/questions/149359/what-is-the-correct-syntax-to-add-cflags-and-ldflags-to-configure) to configure cflags and ldflags.

```bash
export CPATH=$CPATH:/home/chongzhuo/usr/include
export LIBRARY_PATH=$LIBRARY_PATH:/home/chongzhuo/usr/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/chongzhuo/usr/lib
```

### Build YCSB

#### Install Golang

```bash
wget https://mirrors.ustc.edu.cn/golang/go1.19.5.linux-amd64.tar.gz
tar xf go1.19.5.linux-amd64.tar.gz
mv go go-1.19.5
vim ~/.bashrc
# go tools
export GOROOT=$HOME/go-1.19.5
export GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
source ~/.bashrc

# a usefull go proxy
export GOPROXY=https://goproxy.io,direct
```

#### Run YCSB

Before build the go-ycsb, we should check the rocksdb library.

```bash
# test helloworld.cpp
gcc -lrocksdb -x c++ helloworld.cpp

# download and build
git clone https://github.com/pingcap/go-ycsb.git
cd go-ycsb
make -j4
# give it a try
./bin/go-ycsb  --help
```

```bash
# test basic
./bin/go-ycsb load basic -P workloads/workloada

# origin ycsb
# ./bin/ycsb load rocksdb -s -P workloads/workloada -p rocksdb.dir=/mnt/rocksdb
# ./bin/ycsb run rocksdb -s -P workloads/workloada -p rocksdb.dir=/mnt/rocksdb

# go-ycsb
# put rocksdb.dir=/mnt/rocksdb into workload config file
./bin/go-ycsb run rocksdb --threads=4 -P workloads/workload-rocksdb
```

[1] [How do I add SSH Keys to authorized_keys file?](https://askubuntu.com/questions/46424/how-do-i-add-ssh-keys-to-authorized-keys-file)

[2] [How to Access a Remote Server Using a SSH Jump Host](https://www.tecmint.com/access-linux-server-using-a-jump-host/)

[3] [Building GCC 9.2.0 on CentOS 7](https://gist.github.com/nchaigne/ad06bc867f911a3c0d32939f1e930a11)

[4] [Install RocksDB on Ubuntu 20.04 (Focal Fossa)](https://gist.github.com/srimaln91/bea81d8c5ba36a64b0cd1b3b5324f687)

[5] [What is the correct syntax to add CFLAGS and LDFLAGS to "configure"?](https://unix.stackexchange.com/questions/149359/what-is-the-correct-syntax-to-add-cflags-and-ldflags-to-configure)

[6] [A Global Proxy for Go Modules](https://goproxy.io/)

[7] [Install Golang In Your Home Directory And Configure VScode](https://www.pugetsystems.com/labs/hpc/install-golang-in-your-home-directory-and-configure-vscode/)
