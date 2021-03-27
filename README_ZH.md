# Crocodile 分布式任务调度系统

![GitHub Workflow Status](https://img.shields.io/github/workflow/status/labulaka521/crocodile/Build_release)
[![Downloads](https://img.shields.io/github/downloads/labulaka521/crocodile/total.svg)](https://github.com/labulaka521/crocodile/releases)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg?maxAge=2592000)](https://github.com/labulaka521/crocodile/blob/master/LICENSE)
[![Release](https://img.shields.io/github/release/labulaka521/crocodile.svg?label=Release)](https://github.com/labulaka521/crocodile/releases)


[English](./README.md) | 中文 


## Introduction
基于Golang开发的分布式任务调度系统，支持http、golang、python、shell、python3、nodejs、bat等调度任务  

## Screenshot
<details>
<summary>点击我</summary>

![](./screenshot/2.png)
![](./screenshot/8.png)
![](./screenshot/3.png)
![](./screenshot/4.png)
![](./screenshot/5.png)
![](./screenshot/6.png)
![](./screenshot/7.png)
![](./screenshot/1.png)
</details>

```          
                                                  +----------+
        +-------------+                           ||--------||
        ||-----------||                           ||        ||
        ||           ||                           || Worker ||
        ||   调度中心 ||                           ||         ||
        ||           ||      RPC调用(gRPC)        ||---------||
        ||-----------|| +-----------------------> ||-------- ||
        ||-----------||                           ||        ||
        ||           || <-----------------------+ || Worker ||
  +---> ||   调度中心 ||      任务实时日志            ||        ||
任 |    ||           ||                           ||--------||
务 |    ||-----------|| <------+                  ||--------||
完 |     +------------+        |                  ||        ||
成 |         ^    |            |                  || Worker ||
持 |       实|任   |            |数                ||        ||                
久 |       时|务   |获          |据                 +----------+ 
日 |       志|状   |取          |存                
志 |       日|态   |锁          |储
   |        v     v            v
   |      +-------+-+      +----+---+
   |      |         |      |        |
   +----- |  Redis  |      | MySQL  |
          |         |      |        |
          +---------+      +--------+
```

## Features
- 多调度中心、多Worker运行
- 在Web节点对任务进行增加、修改、删除、克隆、运行任务、终止任务等操作
- 实时查看正在运行的任务、任务的实时日志和任务状态
- 多种任务类型:
    - 执行`http`请求任务
    - 运行`shell`、`python`、`golang`、`python3`、`nodejs`代码(当然其他语言也可以支持，如需要请提出)
- 父、子任务:   
    当设置了父任务或者子任务后，先会运行`父任务`->`主任务`->`子任务`，任意任务出错后会立即中断整个流程，还可以设置父任务或子任务`并行`或者`串行`运行
- 调度算法:  
    支持四种调度算法随机、轮训、Worker权重、Worker最少任务数来调用Worker运行任务，
- 自定义报警策略:  
    可以设置当任务`成功`、`失败`、或者`运行完成后`报警给多个用户  
    设置任务的返回码或者返回内容来比较任务的实际返回码或者返回内容是否相同来判断任务运行成功或者，code任务默认为0，http任务默认为200  
- 主机组:  
    一个任务只可以绑定到任意一个主机组，任务的运行会通过任务的路由策略来选取这个主机组中的一个任务来运行任务
- 主机:  
    一个主机组可以绑定多个主机，主机是实际运行任务的节点,注册后调度中心自动发现
- 安全策略  
    证书加密(可关闭)     
    密钥认证(必选，防止未授权Worker节点接入)
- 任务的日志管理，清理日志
- 报警通知支持平台  
    - 邮件  
    - 企业微信  
    - 钉钉  
    - Slack Channel   
    - Telegram Bot
    - WebHook URL
- 详细的操作审计功能  
- 权限控制  
    有三种用户类型
    - 管理员  
        拥有所有操作权限，
    - 普通用户
        可以创建新的任务、主机组,可以对自已创建的任务或主机组进行操作，不能查看审计记录、所有用户
    - 访客  
        只有查看的权限、无任何操作修改权限，不能查看审计记录、所有用户


## Supported platforms
- Linux
- Mac
- Windows


## Quick Start

```
git clone https://github.com/labulaka521/crocodile
cd crocodile
docker-compose up -d
```
然后在浏览器中打开`http://ip:8080`，然后在初始化页面输入一个管理员用户名、用户密码然后点击`开始安装`，安装完成后你就可以进入到系统中去

## Running
- [点击下载](https://github.com/labulaka521/crocodile/releases)后解压
- 安装redis、mysql，然后修改配置文件
- 如果需要开启调度节点之间的证书认证，请生成证书，
    运行生成cert证书的命令
    ```shell
    crocodile cert
    ```
    然后会在当前目录本地生成两个文件`cert.pem`、`key.pem`，将这两个文件保存后，将文件的路径填写值配置文件中,每个节点都需要这两个文件
- 作为一个调度中心来运行  
    可以启动多个调度中心，防止单点故障导致调度挂掉  
    ```shell
    ./crocodile server -c core.toml
    ```
- 作为一个Worker(主机)节点来运行  
    ```shell
    ./crocodile client -c core.toml
    ```
- 查看版本编译信息
    ```
    ./crocodile version
    ```
- [配置报警](https://github.com/labulaka521/crocodile/wiki/%E9%85%8D%E7%BD%AE%E6%8A%A5%E8%AD%A6%E9%80%9A%E7%9F%A5)

## Development
- 前端
    - 安装`yarn`
    - 进入web目录,先下载依赖`yarn`,然后单独运行前端`yarn run dev`
    - 打包至go，执行`make frontrnd`，
- 后端
    - 作为调度中心运行`make runs`
    - 作为Worker节点运行`make runc`
> sql目录也是被打包在`go-bindata`中的，在安装时会从`go-bindata`生成的文件读取sql创建语句，如果修改了sql，就重新执行下`make bin-data`

## Doc
[Wiki](https://github.com/labulaka521/crocodile/wiki/)

## Donate
如果这个项目帮助了你，希望你可以通过支付宝可以捐助

<img src="./screenshot/alipay.jpg" width=100 height=100>


## License
Crocodile is under the MIT license. See the [LICENSE](./LICENSE) file for details.
