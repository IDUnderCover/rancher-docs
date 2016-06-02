# 镜像仓库
--
在 Rancher 中，您可以把 DockerHub、Quay.io 或者任何私有镜像仓库的访问账号添加进来，来访问私有镜像。通过有了访问您的私有镜像库的功能，它是 Rancher 可以去使用您的私有镜像。这样让从一个私有地址中加载一个镜像变得很简单。在 每一个环境中，每一个镜像仓库的地址对一个的访问账号信只能有一条，Rancher 将总是使用最近添加的哪一个。

> **注意:** 目前，镜像仓库只在 **Cattle** 的环境中被支持。

### 添加镜像仓库

点击 **Infrastructure** 然后进入 **Registries** 页面，点击 **Add Registry**。 

对于所有的镜像仓库，您将需要输入 **e-mail address**, **username**, 和 **password** 等信息。对于一个 **Custom** 镜像仓库，您还将需要提供 **registry address**。然后点击 **Create**.

如果您所添加的访问账号信息的地址已经存在了，Rancher 将会开始使用这个新添加的。 

#### 非加密镜像仓库

为了访问一个非加密镜像仓库，你讲需要配置所有主机的 Docker 后台访问。 `DOMAIN` 和 `PORT` 是提供私有镜像仓库服务主机的域名和端口。

> **注意:** 任何石油重启主机的 Docker 服务，您都可能会语调网络代理被卡在 _Starting_ 状态的问题。为了临时解决这个问题，请重启主机。

```bash
# 编辑配置文件 "/etc/default/docker"
$ sudo vi /etc/default/docker
# 在文件结束处添加添加下面这行。如果已经有这行了，则确保把下面的参数加载最后。
$ DOCKER_OPTS="$DOCKER_OPTS --insecure-registry=${DOMAIN}:${PORT}"
# 重启 Docker 服务
$ sudo service docker restart
```

#### 自签名证书

为了与私有镜像仓库使用一个自签发证书。`DOMAIN` 和 `PORT` 是私有镜像仓库服务所在的地址。

```bash
# 从该域名下载证书
$ openssl s_client -showcerts -connect ${DOMAIN}:${PORT} </dev/null 2>/dev/null|openssl x509 -outform PEM >ca.crt
# 复制这个证书文件到对应的目录
$ sudo cp ca.crt /etc/docker/certs.d/${DOMAIN}/ca.crt
# 附加这个证书到一个文件内容末尾
$ cat ca.crt | sudo tee -a /etc/ssl/certs/ca-certificates.crt
# 重启 Docker 服务使之生效
$ sudo service docker restart

```

### 使用镜像仓库

在镜像仓库被创建之后，你将能在启动服务或者容器的时候使用这些私有镜像仓库。镜像名称的语法是与您在`docker run` 命令中是相同的。

`[registry-name]/[namespace]/[imagename]:[version]`

默认情况下，我们假设您总是从`DockerHub`来下拉镜像。

### 编辑镜像仓库

在镜像仓库清单中的

All options for a registry are accessible through the dropdown menu on the right hand side of the listed registry.

For any **Active** registry, you can **Deactivate** the registry, which would prohibit access to the registry. No new containers can be launched with any images in that registry.

For any **Deactivated** registry, you have two options. You can **Activate** the registry, which will allow containers to access images from those registries. Any members of your environment will be able to activate your credential without needing to re-input the password. If you don't want anyone using your credential, you should **Delete** the registry, which will remove the credentials from the environment.

You can **Edit** any registry, which allows you to change the credentials to the registry address. You will not be able to change the registry address. The password is not saved in the "Edit" page, so you will need to re-input it in order to save any changes.

> **Note:** If a registry is invalid (i.e. inactive, removed, or overridden due to a newer credential), any [service]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/adding-services/) or [container]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/containers/) using a private registry image will continue to run. Since the image has already been pulled onto the host, there will be no restrictions on usage of the image regardless of registry permissions. Therefore, any scaling up of services or additional containers using the image will be able to run. Rancher does not check if the credentials are still valid when running containers as we assume that you've already given the host permissions to access the image. 

