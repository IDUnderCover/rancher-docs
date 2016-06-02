## 使用 Route 53 做外部 DNS
---

作为 [Rancher catalog]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/) 的一部分，Rancher 提供了与Amazon Route53 DNS 集成的 DNS 服务。当启动这个服务，有一个 route53 的容器被启动。这个容器会监听Rancer 元数据服务器和事件，会基于元数据的变化更新 DNS 记录，并更新到 Route53 服务中。

### 最佳实践

* 在您部署的 Rancher 的每个环境中，应该有一个容器数量为1的 `route53` 服务。
* 多套 Rancher 部署环境应该不能共享相同的  `hosted zone`。

### 启动 Route53 服务

点击 **Catalog** 标签，您能选中**Route53 DNS Stack**。

输入一个 **名字**，输入关于这个堆栈的 **描述**。

在**Configuration Options**中，你需要提供一下信息：


名称| 值
---|---
AWS Access Key | 您账户AWS API的 Access key 
AWS Secret Key | 您账户AWS API的 Secret key
AWS Region | AWS的 Region 名。我们建议选择离您所地最近的区。更新的信息将会被发送到该 AWS API 端点，Rancher Route53 会随时及时更新 DNS 更新请求。
Hosted Zone | Route53 的 hosted zone。这个 zone 需要提前创建好。

<br>
在填写了这些字段后，点击 **Create**按钮。堆栈将被创建出来带有 `route53` 的服务，而您只需要做启动这个服务即可。


### 使用 Route53 服务

这个 `route53` 服务只会在服务有端口暴露在主机上时，来未知生成 DNS 记录。对于每一个 Rancher 生成的 DNS 记录，它会按下面的形式创建出 fqdn：

```
fqdn=<serviceName>.<stackName>.<environmentName>.<yourHostedZoneName>
```

在 AWS 的 Route 53上，把以 Record Set 的形式被表达：name=fqdn 和 value=[服务被部署的主机的 IP 地址。 Rancher `route53` 服务将只管理以 <environmentName>.<yourHostedZoneName> 结尾的 Record Set，默认的 300 秒。 

一旦在 AWS 的 Route 53里配置了 DNS 记录，被创建的 fqdn 记录将被传回 Rancher，会被配置在 **service.fqdn** 字段。您能通过点击服务的 **View in API**下拉菜单后，搜索找到 **fqdn**找到对应的 fqdn 记录。

当在浏览器中使用 fqdn 时，它会被指向服务中的一个容器中。如果一个服务中容器的 IP 地址会有任何变化，这些变化将也被更新到 AWS Route 53服务中。而从用户使用 fqdn 域名访问服务的角度看，则不会体验到有任何变化。

> **注意:** 在`route53` 服务启动了以后，对于已经部署了的有主机端口的服务将也会得到一个 fqdn。


### 删除 Route53 服务

当从 Rancher 中删除了`route53` 服务之后，Amazon Route 53中的 Record Set 将 **不会** 被自动删除。它们还是需要登录了您的 Amazon AWS 账号之后再手工删除掉。

### 为外部 DNS 使用特定的 IP

默认，Rancher DNS 所使用的主机IP，是注册到 Rancher 中是所指定的IP。这里会有这样一种用例，被配置在 Rancher 中的主机使用私有网络通信，而这个主机将需要通过外部 DNS 使用公网 IP 来对外暴露服务。在这种情况下在你需要制定外部 DNS 所使用的IP，您需要在使用外部 DNS 服务前为主机添加 [主机标签]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/#host-labels) 。

在你启动外部 DNS 服务之前，请添加下面的主机标签。标签的值是 Rancher 的 Route 53
服务会用到的。如果没有设置这个标签的话，Rancher 的 Route53 DNS 服务器就会直接使用主机 IP。

```
io.rancher.host.external_dns_ip=<IP_TO_BE_USED_FOR_EXTERNAL_DNS>
```


