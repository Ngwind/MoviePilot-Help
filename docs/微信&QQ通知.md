# QQ通知

## 安装QQ无头客户端

使用[Lagrange](https://github.com/LagrangeDev/Lagrange.Core/releases)是最方便的部署方式。

除此之外也可以选用这些方案，均具有多端部署的特性：

[NapNekoQQ](https://github.com/NapNeko/NapCatQQ?tab=readme-ov-file)：NapCatQQ 是现代化的基于 NTQQ 的 Bot 协议端实现

[LLOneBot](https://llob.napneko.com/zh-CN/guide/getting-started)：LiteLoaderQQNT 插件，实现 OneBot 11 和 Satori 协议，用于 QQ 机器人开发

### Windows

[LLagrangeDev](https://github.com/LagrangeDev/Lagrange.Core/releases)下载`Lagrange.OneBot_win-x64_net8.0_SelfContained.zip`，双击运行即可，之后参考下文进行配置


### Linux

Linux 安装方法与 Windows 类似，使用多任务管理器创建一个会话，下载对应的二进制文件并运行

如果要使用 Docker 安装 Larganeg，可参考 <https://github.com/LagrangeDev/Lagrange.Core/blob/nightly/Docker.md>


## 配置QQ机器人

首先需要去[QQ官网](https://www.qq.com)注册一个小号，手机可以直接用自己的，同一个手机号可以注册多个QQ号。之后在电脑上登录。



## 配置nonebot机器人

将本项目中的[nonebot-plugin](https://github.com/Putarku/MoviePilot-Help/tree/main/nonebot-plugin)文件夹的内容下载到本地，修改`docker-compose.yml`中的MP配置信息</br>
之后在[nonebot-plugin](https://github.com/Putarku/MoviePilot-Help/tree/main/nonebot-plugin)文件夹中打开命令行  运行`docker-compose up -d`，即可启动nonebot机器人

```bash
cd ./nonebot-plugin
vi docker-compose.yml
docker-compose up -d
```

按情况在`Lagrange`的`appsettings.json`中更改`ReverseWebSocket`项，本例中只需要填写ip和对应端口（默认8083），这样`Larganeg`就可以与`nonebot`进行通信了。


<Details>
<Summary>Lagrange的示例appsettings.json</Summary>

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "SignServerUrl": Find SignServer In Some Loc,
  "Account": {
    "Uin": 0,
    "Password": "",
    "Protocol": "Linux",
    "AutoReconnect": true,
    "GetOptimumServer": true
  },
  "Message": {
    "IgnoreSelf": true,
    "StringPost": false
  },
  "QrCode": {
    "ConsoleCompatibilityMode": false
  },
  "Implementations": [
    {
      "Type": "ReverseWebSocket",
      "Host": 你的IP,
      "Port": 8083,
      "Suffix": "/onebot/v11/ws",
      "ReconnectInterval": 5000,
      "HeartBeatInterval": 5000,
      "HeartBeatEnable": true,
      "AccessToken": ""
    },
    {
      "Type": "Http",
      "Host": "*",
      "Port": 8088,
      "AccessToken": ""
    }
  ]
}
```
</details>


## 设置MP的通知信息
 <div align=center> <img src="https://github.com/Putarku/MoviePilot-Help/raw/main/img/QQ_1726668218021.png" width="600"> </div>
 
 其中服务器地址为上面`Larganeg`的ip，端口默认为8088，可在上面的`appsettings.json`中修改。


## 使用

此时私聊或是群聊中发送`/sub 片名`即可触发查询并添加订阅。

 <div align=center> <img src="https://github.com/Putarku/MoviePilot-Help/raw/main/img/QQ20240922-163922.png" width="600"> </div>


<br>

## 配置MP插件

消息聚合插件在第三方插件市场：https://github.com/hotlcc/MoviePilot-Plugins-Third

 <div align=center> <img src="https://github.com/Putarku/MoviePilot-Help/raw/main/img/QQ_1726668218021.png" width="600"> </div>
填写上面机器人的http地址和端口，以及想要通知的QQ号和QQ群号，同时点击`配置消息模板`，将下面的模板复制进去，保存即可。

```block
${render_image(image)}
${title}
${render_text(text)}
<%!
def render_image(image):
    if image:
        return f"[CQ:image,file={image}]"
    return ""

def render_text(text):
    if text is not None:
        return text
    return ""
%>
```

<br>

# 微信通知

### 如何配置企业微信通知
  
  1.参见[此教程](https://pt-helper.notion.site/50a7b44e255d40109bd7ad474abfeba5)
  
  <br>

### 建立企业微信的代理服务器
  

首先需要先准备一个具有固定公网地址的服务器，例如VPS，之后在该服务器上搭建代理服务。搭建方式可以有以下两种，两种任选其一即可

 > #### 1、使用[`caddy`](https://github.com/caddyserver/caddy)搭建

  1. 从 https://github.com/caddyserver/caddy/releases
下载自己对应系统的版本，例如 AMD64 下载`caddy_2.7.5_linux_amd64.tar.gz`
  1. 解压得到 `caddy` 文件 上传到`/usr/local/bin` 目录下，注意设置权限 `0755`
  2. 在任意目录新建 `Caddyfile` 文件(例如`/usr/local/caddy`) ，注意设置权限 `0755`，文
件内容如下
```yaml
:3000
reverse_proxy https://qyapi.weixin.qq.com {
header_up Host {upstream_hostport}
}
```
  1. SSH 控制台 cd 到 `Caddyfile` 文件的目录(例如`/usr/local/caddy`)
  2. 输入 caddr start 启动完成，在防火墙中放行3000端口
  3.  NasTools / MoviePilot 设置微信的代理 IP 地址为 `http://你的服务器ip/域名:3000`

<br>

 > #### 2、使用[ddsderek/wxchat](https://hub.docker.com/r/ddsderek/wxchat)docker镜像搭建

```yaml
version: '3.3'
services:
    wxchat:
        container_name: wxchat
        restart: always
        ports:
            - '3000:80'
        image: 'ddsderek/wxchat:latest'
```
```yaml
docker run -d \
    --name wxchat \
    --restart=always \
    -p 3000:80 \
    ddsderek/wxchat:latest
```
搭建完成后，在防火墙中放行3000端口，并在NasTools / MoviePilot 设置微信的代理 IP 地址为 `http://你的服务器ip/域名:3000`

<br>

### 配置企业微信时提示“回调失败”
  

 1.在企业微信的填写的地址可以有两种方式

 ①`http://ip:端口/api/v1/message/?token=moviepilot`

 ②`http://ip:端口/api/v1/message/`

 如果自行配置了`API_TOKEN`的值，那么就需要在地址后面补上`?token=moviepilot`。如果`API_TOKEN`为默认值，那么两种填写方式均可。

 2.确认在手机打开流量时，直接打开`http://ip:端口`，可以直接访问MoviePilot的网页。

 3.微信不支持ipv6,因此如果域名是使用ipv6解析的时候，也会导致不通过。如果没有ipv4的公网ip，建议使用内网穿透。

 <br>

 ### 企业微信部署后不显示菜单

如果是沿用nastool的代理服务器配置，需要在`nginx`的配置文件中额外加入下列代码，才能自动生成菜单。

```
location  /cgi-bin/menu/create {
    proxy_pass https://qyapi.weixin.qq.com;
}
```

 <br>



