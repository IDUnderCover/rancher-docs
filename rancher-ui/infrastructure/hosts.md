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

* [添加自定义主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/custom/)
* [添加 Amazon EC2 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/amazon/)
* [添加 Azure 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/azure/)
* [添加 DigitalOcean 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/digitalocean/)
* [添加 Exoscale 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/exoscale/)
* [添加 Packet 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/packet/)
* [添加 Rackspace 主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/rackspace/)
* [添加其他驱动的主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/other/)

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
# Adding one host label to the rancher/agent command
$  sudo docker run -e CATTLE_HOST_LABELS='foo=bar' -d --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
    http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>

# Adding more than one host label requires joining the additional host labels with an `&`
$  sudo docker run -e CATTLE_HOST_LABELS='foo=bar&hello=world' -d --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
    http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>
```

<br>

> **注意：** The `rancher/agent` version is correlated to the Rancher server version. You will need to check the custom command to get the appropriate tag for the version to use. 

#### Automatically Applied Host Labels

Rancher automatically creates host labels related to linux kernel version and Docker Engine version of the host. 

Key | Value | Description
----|----|----
`io.rancher.host.linux_kernel_version` | Linux Kernel Version on Host (e.g, `3.19`) |  Version of the Linux kernel running on the host
`io.rancher.host.docker_version` | Docker Version on the host (e.g. `1.10.3`) | Docker Engine Version on the host


### Hosts behind a Proxy
To support hosts behind a proxy, you'll need to edit the Docker daemon to point to the proxy. The detailed instructions are listed within our [adding custom host page]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/custom/#hosts-behind-a-proxy).

<a id="machine-config"></a>

### Accessing hosts from the Cloud Providers 
After Rancher launches the host, you may want to be able to access the host. We provide all the certificates generated when launching the machine in an easy to download file. Click on **Machine Config** in the host's dropdown menu. It will download a tar.gz file that has all the certificates.

To SSH into your host, go to your terminal/command prompt. Navigate to the folder of all the certificates and ssh in using the `id_rsa` certificate.

```bash
$ ssh -i id_rsa root@<IP_OF_HOST>
```

## Cloning a Host
---

Since launching hosts on cloud providers requires using an access key, you might want to easily create another host without needing to input all the credentials again. Rancher provides the ability to clone these credentials to spin up a new host. Select **Clone** from the host's drop down menu. It will bring up an **Add Host** page with the credentials of the cloned host populated.

## Editing Hosts
---

The options for what can be done to a host are located in the host's dropdown. From the **Infrastructure** -> **Hosts** page, the dropdown icon will appear when you hover over the host. If you click on the host name to view more details of a host, the dropdown icon is located in the upper right corner of the page. It's located next to the State of the host.

If you select **Edit**, you can update the name, description or labels on the host. 

### Deactivating/Activating Hosts

Deactivating the host will put the host into an _Inactive_ state. In this state, no new containers can be deployed. Any active containers on the host will continue to be active and you will still have the ability to perform actions on these containers (start/stop/restart). The host will still be connected to the Rancher server. Select **Deactivate** from the host's dropdown menu.

When a host is in the _Inactive_ state, you can bring the host back into an _Active_ state by clicking on **Activate** from the host's dropdown menu.

> **Note:** If a host is down in Rancher (i.e. in `reconnecting` or `inactive` state), you will need to implement a [health check]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/health-checks/) in order for Rancher to launch the containers on your service on to a different host.

## 删除主机
---

为了从服务端删除主机，你需要在下拉菜单中操作一系列的步骤。

选择 **Deactivate**。当主机完成停用，主机将显示 _Inactive_ 状态。选择 **Delete**。服务端将开始将主机从 Rancher 服务端移除的过程。在它完成删除后首先显示的状态将会是 _Removed_。在从UI中立即消失前，将会继续以便完成移除过程，并进入 _Purged_ 状态。

如果使用 Rancher 在云提供商上创建的主机，主机将会在云提供商处删除。如果主机是通过使用 [自定义命令]()添加的，主机将会保留在云提供商处。

> **注意：** 对于自定义主机，所有容器，包括 Rancher 代理端将会继续保留在主机上。

## 删除 Rancher 外的主机
---

如果你的主机在 Rancher 外被删除了，Rancher 服务器将会继续展示此主机直到它被移除。最终，这些主机将显示为处于 _Reconnecting_ 状态但无法重新连接。你能够 **Delete** 这些主机用于从UI上移除他们。


