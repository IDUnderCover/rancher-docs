# 从主机开始

## 从主机开始
---
在 Rancher 中，我们在UI上提供了很简单的指导，来将那些直接支持的云提供商所提供的主机添加进来，同时也提供了指导，在你的云提供商不支持的时候,添加你拥有的主机。在 **Infrastructure** 中的 **Hosts** 页，点击 **Add Host** 。


### 主机要求

* 任何现代的 Linux 发行版，支持 Docker 1.10.3。[RancherOS](http://docs.rancher.com/os/), Ubuntu, RHEL/CentOS 7 经过了更严格的测试。
* 1GB RAM 
* 建议 CPU w/ AES-NI

### 主机如何工作?

在 rancher 代理端在主机上启动后，主机就连上了 Rancher 服务端。注册令牌，就是 **Add Host** -> **Custom** 页面中的长URL，用来帮助 rancher 代理端首次连上服务端。在连接中，将会在 Rancher 服务端生成一个代理端帐号和 API 密钥对。密钥对用来作为后来的所有通讯，使用如其他类型的帐号如环境 API 密钥，相同的认证与授权逻辑。

设计如此是因为代理端是不可信的，因为他运行在外部的和（对服务端）有潜在敌意的硬件上。代理端帐号仅能在 API 中访问他们所需的资源，对事件的回复会被检查确保事件确实发送给此代理端等。由于没有反方向的为代理端来验证主机，因此你可以设置 TLS 并且证书将会被验证。

IPSec 密钥是每个 [环境]()独有的。他由服务端生成，存储在数据库中，并作为代理端注册 API 密钥对的一部分发送给主机。连接是在主机间点对点并 AES 加密的，其中加密能被大多数的现代 CPU 加速。

<a id="addhost"></a>

## 添加主机
---

你第一次添加主机时，你可能被要求设置好 [主机注册]()。设置将检测何种DNS名称或ip地址及端口用来使你的主机链接到 Rancher API。默认的情况下，我们选择管理服务端 IP 和端口 `8080`。 如果你选择改变地址，请确保指定端口用来连接到 Rancher API。任何时候你都能够更新 [主机注册]()。在设置好你的主机注册后，点击 **Save**。

我们支持直接从云提供商添加主机，或添加已经被配置好了的主机。对于云提供商，我们使用 `docker-machine` 进行配置，并支持任何 `docker-machine` 支持的镜像。

选择你想添加的何种主机类型：

* [添加自定义主机]()
* [添加 Amazon EC2 主机]()
* [添加 Azure 主机]()
* [添加 DigitalOcean 主机]()
* [添加 Exoscale 主机]()
* [添加 Packet 主机]()
* [添加 Rackspace 主机]()
* [添加其他驱动的主机]()

当一台主机加入 Rancher，rancher 代理端将会在主机上启动。Rancher 将会自动拉取 `rancher/agent` 的正确镜像版本标签并运行所需的版本。对于每个 Rancher 服务端版本，代理端版本都被明确标记。

<a id="labels"></a>

### 主机标签

对于每个主机，你能添加标签来协助组织你的主机。当启动 rancher/agent 容器时，标签作为环境变量被添加。UI中的主机标签将会是 key/value 对并且key都需要是唯一标识。如果你添加了两个相同的 key 带有不同 value，我们将使用后输入的 value 作为 key/value 对。

通过对主机添加标签，你能在[调度服务/负载均衡/服务]()中使用标签，创建对你的[服务]()运行的主机白名单或黑名单。

如果你计划使用 [外部 DNS 服务]()，并需要 [编辑 DNS 记录来使用非主机IP的其他IP]()，你需要在主机上包含此标签 `io.rancher.host.external_dns_ip=<IP_TO_BE_USED_FOR_EXTERNAL_DNS>` 。主机标签能在注册主机或在主机添加进 Rancher 后添加，但是应该在外部 DNS 服务开启前添加到主机。标签的值将会在编辑外部 DNS 服务的规则时被使用。

当使用UI添加不同云提供商的主机时，rancher/agent 命令会自动为你执行并带有通过UI添加的主机标签。

当添加自定义主机，你能通过UI添加标签并且它将自动添加带有 key/value 对的环境变量(`CATTLE_HOST_LABELS`)到UI界面的命令中。

_例子_

```bash
# 添加一个主机标签到 rancher/agent 命令中
$  sudo docker run -e CATTLE_HOST_LABELS='foo=bar' -d --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
    http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>

# 添加多于一个主机标签需要使用 `&` 连接多余的主机标签
$  sudo docker run -e CATTLE_HOST_LABELS='foo=bar&hello=world' -d --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
    http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>
```

<br>

> **注意：** `rancher/agent` 的版本与 Rancher 服务端的版本相关联。你需要检查自定义命令以便获取所使用版本的正确标识。

#### 自动实现的主机标签

Rancher 自动创建关于 Linux 内核版本和 Docker Engine 版本的主机标签。

Key | Value | 描述
----|----|----
`io.rancher.host.linux_kernel_version` | 主机的 Linux 内核版本(例如： `3.19`) |  主机上运行的 Linux 内核的版本
`io.rancher.host.docker_version` | 主机的 Docker 版本 (例如： `1.10.3`) | 主机上的 Docker Engine 版本


### 代理后的主机
为了支持代理后面的主机，你需要编辑 Docker daemon 来指向代理。详细指导列在我们的 [添加自定义主机 ]()页面

<a id="machine-config"></a>

### 访问来自于云提供商的主机
在 Rancher 启动主机后，你可能需要访问主机。我们提供了所有在启动设备时生成的证书，可通过下载方式获取。在主机的下拉菜单中点击 **Machine Config**。那将会下载一个包含所有证书的 tar.gz 文件。

为了 SSH 进入你的主机，进入你的终端/命令行。进入所有证书存放的文件夹，然后通过使用 `id_rsa` 证书以便 ssh 进入主机。

```bash
$ ssh -i id_rsa root@<IP_OF_HOST>
```

## 克隆主机
---

由于启动云提供商上的主机需要使用 access key，你可能想要更简单的创建另一个主机而不需要再次输入所有的证书。Rancher 提供了通过克隆这些证书来启动一个新主机的功能。从主机的下拉菜单中选择 **Clone**。那将会引向 **Add Host** 页面并带有克隆主机的所使用的证书。

## 编辑主机
---

所有能对主机做的操作选项都位于主机的下拉菜单中。在 **Infrastructure** -> **Hosts** 页面中，下拉标识将会在你鼠标放置在主机上时出现。如果你点击主机名字来查看主机的更多信息，下拉标识将会出现在页面的右上角。它将会位于主机状态的旁边。

如果你选择 **Edit**，你能更新名字，描述或主机的标签。


### 停用/激活主机

停用主机将使主机进入 _Inactive_ 状态。在此状态中，将不会部署新的容器。任何此主机上的活动的容器将会保持激活，并且你仍然能够对这些容器执行操作（开始/停止/重启）。主机将仍然连接到 Rancher 服务端。从主机的的下拉菜单中选择 **Deactivate**。

当主机处于 _Inactive_ 状态，你能通过点击主机下拉菜单中的 **Activate** 使主机重回 _Active_ 状态。

> **注意：** 如何在 Rancher 中一台主机失效 (例如 处于 `reconnecting` 或 `inactive` 状态)，你将需要执行 [健康检查]() 以便 Rancher 在其他不同的主机上启动你服务中的容器。

## 删除主机
---

为了从服务端删除主机，你需要在下拉菜单中操作一系列的步骤。

选择 **Deactivate**。当主机完成停用，主机将显示 _Inactive_ 状态。选择 **Delete**。服务端将开始将主机从 Rancher 服务端移除的过程。在它完成删除后首先显示的状态将会是 _Removed_。在从UI中立即消失前，将会继续以便完成移除过程，并进入 _Purged_ 状态。

如果使用 Rancher 在云提供商上创建的主机，主机将会在云提供商处删除。如果主机是通过使用 [自定义命令]()添加的，主机将会保留在云提供商处。

> **注意：** 对于自定义主机，所有容器，包括 Rancher 代理端将会继续保留在主机上。

## 删除 Rancher 外的主机
---

如果你的主机在 Rancher 外被删除了，Rancher 服务器将会继续展示此主机直到它被移除。最终，这些主机将显示为处于 _Reconnecting_ 状态但无法重新连接。你能够 **Delete** 这些主机用于从UI上移除他们。


