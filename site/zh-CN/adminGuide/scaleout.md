---
id: scaleout.md
related_key: scale Milvus cluster
summary: Learn how to manually or automatically scale out and scale in a Milvus cluster.
---

# Milvus集群扩缩容

{{fragments/translation_needed.md}}

Milvus可以对其组件横向扩缩容。也就是说，你可以按需增加(或减少)任何类型的节点数量。 

本文介绍如何对Milvus集群扩容和缩容。我们假设您已经[安装了Milvus集群](install_cluster-helm.md)。除此之外，建议您先熟悉[Milvus架构](architecture_overview.md)，再对集群进行扩缩容。

本教程对三个查询节点扩容、缩容作为样例。如果要对其他类型的节点扩容，只需要在命令行中以对应的节点类型替换`queryNode`即可。

## 什么是水平扩缩容?

水平扩缩容包括扩容和缩容。

### 扩容 
扩容即增加集群中的节点数量。不同于按比例放大，扩容并不需要在一个节点上放置更多的资源，而是通过增加节点数量，来水平的扩展集群。

![水平扩容](../../../assets/scale_out.jpg)

![按比例放大](../../../assets/scale_up.jpg)

根据[Milvus架构](architecture_overview.md)，无状态的工作节点包括查询节点、数据节点、索引节点、和代理。因此您可以对这些节点扩容来满足您的场景需要。您可以对Milvus集群手动扩容，也可以自动扩容。

通常来说，当Milvus集群过负载时，您需要对其进行扩容。下面是一些您需要扩容的典型场景：
- CPU和内存持续的高使用率。
- 需要更高的查询吞吐。
- 需要更快建立索引。
- 需要处理更大的数据集。
- 需要保证高可用。


### 缩容
缩容即减少集群中节点个数。通常来说，当集群利用率较低时，您需要对集群缩容。以下是一些典型的需要缩容的场景：
- CPU和内存持续的低使用率。
- 查询吞吐需求量小。
- 不需要快速建立索引。
- 需要处理的数据集小。

<div class="注意事项">
我们不推荐大幅度减少工作节点数量。例如，如果集群中有5个节点，那么我们推荐每次减少一个节点来保证服务的可用性。如果减少一个节点后服务依旧可用，您可以继续尝试减少节点。
</div>

## 前期准备

执行 `kubectl get pods` 获取Milvus集群中的组件列表，以及各组件的状态。

```
NAME                                            READY   STATUS       RESTARTS   AGE
my-release-etcd-0                               1/1     Running      0          1m
my-release-milvus-datacoord-7b5d84d8c6-rzjml    1/1     Running      0          1m
my-release-milvus-datanode-665d4586b9-525pm     1/1     Running      0          1m
my-release-milvus-indexcoord-9669d5989-kr5cm    1/1     Running      0          1m
my-release-milvus-indexnode-b89cc5756-xbpbn     1/1     Running      0          1m
my-release-milvus-proxy-7cbcc8ffbc-4jn8d        1/1     Running      0          1m
my-release-milvus-pulsar-6b9754c64d-4tg4m       1/1     Running      0          1m
my-release-milvus-querycoord-75f6c789f8-j28bg   1/1     Running      0          1m
my-release-milvus-querynode-7c7779c6f8-pnjzh    1/1     Running      0          1m
my-release-milvus-rootcoord-75585dc57b-cjh87    1/1     Running      0          1m
my-release-minio-5564fbbddc-9sbgv               1/1     Running      0          1m 
```

<div class="注意">
Milvus仅支持增加工作节点，不支持增加协调节点。 
</div>

## 缩放Milvus集群 

您既可以手动对集群扩缩容，也可以设置自动扩缩容。如果开启了自动扩缩容，当CPU和内存使用量达到您所设置的阈值时，就会自动扩容或缩容。


### 手动扩缩容

#### 扩容

执行 `helm upgrade my-release milvus/milvus --set queryNode.replicas=3 --reuse-values` 对查询节点手动扩容。

如果成功，查询节点上将有3个运行着的pods，如下所示。

```
NAME                                            READY   STATUS    RESTARTS   AGE
my-release-etcd-0                               1/1     Running   0          2m
my-release-milvus-datacoord-7b5d84d8c6-rzjml    1/1     Running   0          2m
my-release-milvus-datanode-665d4586b9-525pm     1/1     Running   0          2m
my-release-milvus-indexcoord-9669d5989-kr5cm    1/1     Running   0          2m
my-release-milvus-indexnode-b89cc5756-xbpbn     1/1     Running   0          2m
my-release-milvus-proxy-7cbcc8ffbc-4jn8d        1/1     Running   0          2m
my-release-milvus-pulsar-6b9754c64d-4tg4m       1/1     Running   0          2m
my-release-milvus-querycoord-75f6c789f8-j28bg   1/1     Running   0          2m
my-release-milvus-querynode-7c7779c6f8-czq9f    1/1     Running   0          5s
my-release-milvus-querynode-7c7779c6f8-jcdcn    1/1     Running   0          5s
my-release-milvus-querynode-7c7779c6f8-pnjzh    1/1     Running   0          2m
my-release-milvus-rootcoord-75585dc57b-cjh87    1/1     Running   0          2m
my-release-minio-5564fbbddc-9sbgv               1/1     Running   0          2m
```

#### 缩容

执行 `helm upgrade my-release milvus/milvus --set queryNode.replicas=1 --reuse-values` 对查询节点手动缩容.

如果成功了，查询节点上运行着的3个pods将缩减为1个，如下所示：

```
NAME                                            READY   STATUS    RESTARTS   AGE
my-release-etcd-0                               1/1     Running   0          2m
my-release-milvus-datacoord-7b5d84d8c6-rzjml    1/1     Running   0          2m
my-release-milvus-datanode-665d4586b9-525pm     1/1     Running   0          2m
my-release-milvus-indexcoord-9669d5989-kr5cm    1/1     Running   0          2m
my-release-milvus-indexnode-b89cc5756-xbpbn     1/1     Running   0          2m
my-release-milvus-proxy-7cbcc8ffbc-4jn8d        1/1     Running   0          2m
my-release-milvus-pulsar-6b9754c64d-4tg4m       1/1     Running   0          2m
my-release-milvus-querycoord-75f6c789f8-j28bg   1/1     Running   0          2m
my-release-milvus-querynode-7c7779c6f8-pnjzh    1/1     Running   0          2m
my-release-milvus-rootcoord-75585dc57b-cjh87    1/1     Running   0          2m
my-release-minio-5564fbbddc-9sbgv               1/1     Running   0          2m
```

### 自动扩缩容
执行如下语句，开启对查询节点自动扩缩容。您还需要设定触发扩缩容的CPU和内存。

```
helm upgrade my-release milvus/milvus --set queryNode.autoscaling.enabled=true --reuse-values
```

## 接下来

- 如果您想了解如何监控Milvus服务以及创建告警:
  - 学习[利用Kubernetes中的Prometheus Operator监控Milvus 2.0](monitor.md)

- 如果您已具备在云上部署集群的条件:
  - Learn how to [Deploy Milvus on AWS with Terraform and Ansible](aws.md)
  - Learn how to [Deploy Milvus on Amazon EKS with Terraform](eks.md)
  - Learn how to [Deploy Milvus Cluster on GCP with Kubernetes](gcp.md)
  - Learn how to [Deploy Milvus on Microsoft Azure With Kubernetes](azure.md)

- 如果您在找如何部署资源：
  - [在Kubernetes上部署资源](allocate.md#standalone)

