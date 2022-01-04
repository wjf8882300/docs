---
title: 以太坊geth安装
date: 2022-01-04 14:43:04
tags: 以太坊
categories: 区块链
---

geth是以太坊的客户端，可用于跟以太坊的链进行交互。它可以同步区块到本地，也可以发布交易到区块上，甚至可以挖矿。
对于我来说，我用geth来获取以太坊交易信息，用户余额，部署合约，调用合约等。

### 1.镜像
以太坊 网站 列出以下可用的镜像及其说明。
ethereum/client-go:latest是Geth的最新开发版本
ethereum/client-go:stable是Geth的最新稳定版本
ethereum/client-go:{version}是Geth在特定版本号上的稳定版本
ethereum/client-go:release-{version}是Geth在特定版本系列上的最新稳定版本
从容器运行时，默认情况下打开以下端口。
8545 TCP，由基于HTTP的JSON RPC API使用
8546 TCP，由基于WebSocket的JSON RPC API使用
30303 TCP和UDP，由运行网络的P2P协议使用
30304 UDP，由P2P协议的新对等发现覆盖使用

### 2.docker安装
安装rinkeby测试链为例
```
mkdir -p /data/rinkeby
cd /data
 
docker run -itd \
--restart=unless-stopped \
-v /etc/localtime:/etc/localtime \
-v /etc/timezone:/etc/timezone \
--name rinkeby-eth \
-v $(pwd)/rinkeby:/root/.ethereum/rinkeby \
-p 8545:8545 -p 30303:30303 \
ethereum/client-go \
--rinkeby \
--rpcapi db,eth,net,web3,personal,web3,txpool \
--syncmode=fast \
--rpc --rpcaddr 0.0.0.0 \
--cache 2048 \
--maxpeers 30 \--allow-insecure-unlock
```

### 3.执行geth
```

docker exec -it rinkeby-eth bash
geth attach /root/.ethereum/rinkeby/geth.ipc
 
# 查看区块同步状态
>eth.syncing
```

#### 3.1 geth命令
- eth.syncing 如果返回的是false，证明同步完成了，可以使用当前节点。否则会返回同步状态
- currentBlock为当前下载到的区块高度，请注意，下载块不等于同步数据了，下载块是一个简单快速的过程，只验证相关的工作量证明，在下载块的同时geth也在一直下载所有的区块数据，当数据下载完成后届时才会处理曾经发生过的所有交易，重新组装整个链。
- txpool.content 获取池内交易详情
- txpool.inspect 获取池内交易概况
- txpool.status 获取交易状态

#### 3.2清空池
有时会出现所有交易都被阻塞的情况，可以手工清空池，然后重启服务，重新同步。
```
 rm -rf /root/.ethereum/rinkeby/transactions.rlp
``` 