# tendermint同步流程

本文介绍一个全新的tendermint节点，如何获取有效的块进行同步，以及同步流程和原理。涉及一些相关的术语需要简单了解一下。

## node类型
我们知道tendermint有validator node，full node和sentry node，这些node是干什么的，有什么作用。
从功能上分为sentry node和validator node，sentry node是为保护validator node不被攻击，可以视为validator node的代理，一个validator node可以有多个sentry node。其实还有seed node，种子节点，seed node可以是外层sentry node，最好不要是直接连validator的sentry node。它负责收集和发送p2p node节点信息。当然还有一般节点，就是我们开发或者使用tendermint链而进行同步的node，由于没有特别功能，所以也没有特别命名。

从内容上还可以分为full node和lite node，字面可以理解我全节点和轻节点，全节点包括链上的所有数据，情节点有部分数据。上面提到的validator node，sentry node都是full node，seed node最好也是全节点，轻节点只适用于一般使用节点，如手机端。

## peer类型
从配置的角度，还可以分为不同的peer类型。
persistent peer，可以理解为永久peer，就是当这个peer的链接断开后，会持续尝试链接该peer。sentry node和validator node直接需要配置成persistent peer。
private peer，私有peer，就是该peer和该peer传过来的peer不会对外广播。sentry node中，需要把validator node配置成private peer。
external peer，告诉外界自己peer的地址。


## 同步方式

* fast sync

  fast sync直接从peer获取到块的信息，然后验证块的有效性，执行块的交易，最后写入state。
  
* 非fast sync

  非fast sync的同步方式需要经过prevote-->precommit，然后验证块，执行块的交易，最后写入state。

## 同步流程
首先我们需要拿到要同步的链的创世块配置信息（genesis.json），该配置文件包括许多配置项，里面有初始的验证人，可以唯一确定一条链，然后配置peer或者seed，seed用来获取peer，可以从peer获取块信息，然后通过验证人进行验证，最后写入状态，从而可以验证后续的块，这样通过一步一步获取到的块即为可信的块。
