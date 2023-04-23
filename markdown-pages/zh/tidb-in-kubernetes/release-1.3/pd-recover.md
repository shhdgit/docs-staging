---
title: 使用 PD Recover 恢复 PD 集群
summary: 了解如何使用 PD Recover 恢复 PD 集群。
---

# 使用 PD Recover 恢复 PD 集群

PD Recover 是对 PD 进行灾难性恢复的工具，用于恢复无法正常启动或服务的 PD 集群。该工具的详细介绍参见 [TiDB 文档 - PD Recover](https://docs.pingcap.com/zh/tidb/stable/pd-recover)。本文档介绍如何下载 PD Recover 工具，以及如何使用该工具恢复 PD 集群。

## 下载 PD Recover

1. 下载 TiDB 官方安装包：

    
    ```shell
    wget https://download.pingcap.org/tidb-community-toolkit-${version}-linux-amd64.tar.gz
    ```

    `${version}` 是 TiDB 集群版本，例如，`v6.1.0`。

2. 解压安装包：

    
    ```shell
    tar -xzf tidb-community-toolkit-${version}-linux-amd64.tar.gz
    tar -xzf tidb-community-toolkit-${version}-linux-amd64/pd-recover-${version}-linux-amd64.tar.gz
    ```

    `pd-recover` 在当前目录下。

## 使用 PD Recover 恢复 PD 集群

本小节详细介绍如何使用 PD Recover 来恢复 PD 集群。

### 第 1 步：获取 Cluster ID

使用以下命令获取 PD 集群的 Cluster ID：


```shell
kubectl get tc ${cluster_name} -n ${namespace} -o='go-template={{.status.clusterID}}{{"\n"}}'
```

示例：

```
kubectl get tc test -n test -o='go-template={{.status.clusterID}}{{"\n"}}'
6821434242797747735
```

### 第 2 步：获取 Alloc ID

使用 `pd-recover` 恢复 PD 集群时，需要指定 `alloc-id`。`alloc-id` 的值是一个比当前已经分配的最大的 `Alloc ID` 更大的值。

1. 参考[访问 Prometheus 监控数据](monitor-a-tidb-cluster.md#访问-prometheus-监控数据)打开 TiDB 集群的 Prometheus 访问页面。

2. 在输入框中输入 `pd_cluster_id` 并点击 `Execute` 按钮查询数据，获取查询结果中的最大值。

3. 将查询结果中的最大值乘以 `100`，作为使用 `pd-recover` 时指定的 `alloc-id`。

### 第 3 步：恢复 PD 集群 Pod

> **警告：**
>
> 通过新建 PD 方式来恢复集群，会丢失之前 PD 已生效的所有配置信息。

1. 删除 PD 集群 Pod。

    通过如下命令设置 `spec.pd.replicas` 为 `0`：

    
    ```shell
    kubectl patch tc ${cluster_name} -n ${namespace} --type merge -p '{"spec":{"pd":{"replicas": 0}}}'
    ```

    由于此时 PD 集群异常，TiDB Operator 无法将上面的改动同步到 PD StatefulSet，所以需要通过如下命令设置 PD StatefulSet `spec.replicas` 为 `0`：

    
    ```shell
    kubectl patch sts ${cluster_name}-pd -n ${namespace} -p '{"spec":{"replicas": 0}}'
    ```

    通过如下命令确认 PD Pod 已经被删除：

    
    ```shell
    kubectl get pod -n ${namespace}
    ```

2. 确认所有 PD Pod 已经被删除后，通过如下命令删除 PD Pod 绑定的 PVC：

    
    ```shell
    kubectl delete pvc -l app.kubernetes.io/component=pd,app.kubernetes.io/instance=${cluster_name} -n ${namespace}
    ```

3. PVC 删除完成后，扩容 PD 集群至一个 Pod。

    通过如下命令设置 `spec.pd.replicas` 为 `1`：

    
    ```shell
    kubectl patch tc ${cluster_name} -n ${namespace} --type merge -p '{"spec":{"pd":{"replicas": 1}}}'
    ```

    由于此时 PD 集群异常，TiDB Operator 无法将上面的改动同步到 PD StatefulSet，所以需要通过如下命令设置 PD StatefulSet `spec.replicas` 为 `1`：

    
    ```shell
    kubectl patch sts ${cluster_name}-pd -n ${namespace} -p '{"spec":{"replicas": 1}}'
    ```

    通过如下命令确认 PD 已经启动：

    
    ```shell
    kubectl logs -f ${cluster_name}-pd-0 -n ${namespace} | grep "Welcome to Placement Driver (PD)"
    ```

### 第 4 步：使用 PD Recover 恢复 PD 集群

1. 拷贝 `pd-recover` 到 PD pod：

    
    ```shell
    kubectl cp ./pd-recover ${namespace}/${cluster_name}-pd-0:./
    ```

2. 使用 `pd-recover` 恢复 PD 集群：

    
    ```shell
    kubectl exec ${cluster_name}-pd-0 -n ${namespace} -- ./pd-recover -endpoints http://127.0.0.1:2379 -cluster-id ${cluster_id} -alloc-id ${alloc_id}
    ```

    `${cluster_id}` 是[获取 Cluster ID](#第-1-步获取-cluster-id) 步骤中获取的 Cluster ID，`${alloc_id}` 是[获取 Alloc ID](#第-2-步获取-alloc-id) 步骤中获取的 `pd_cluster_id` 的最大值再乘以 `100`。

    `pd-recover` 命令执行成功后，会打印如下输出：

    ```shell
    recover success! please restart the PD cluster
    ```

### 第 5 步：重启 PD Pod

1. 删除 PD Pod：

    
    ```shell
    kubectl delete pod ${cluster_name}-pd-0 -n ${namespace}
    ```

2. 通过如下命令确认 Cluster ID 为[获取 Cluster ID](#第-1-步获取-cluster-id) 步骤中获取的 Cluster ID：

    
    ```shell
    kubectl -n ${namespace} exec -it ${cluster_name}-pd-0 -- wget -q http://127.0.0.1:2379/pd/api/v1/cluster
    kubectl -n ${namespace} exec -it ${cluster_name}-pd-0 -- cat cluster
    ```

### 第 6 步：扩容 PD 集群

通过如下命令设置 `spec.pd.replicas` 为期望的 Pod 数量：


```shell
kubectl patch tc ${cluster_name} -n ${namespace} --type merge -p '{"spec":{"pd":{"replicas": $replicas}}}'
```

### 第 7 步：重启 TiDB 和 TiKV

使用以下命令重启 TiDB 和 TiKV 实例：


```shell
kubectl delete pod -l app.kubernetes.io/component=tidb,app.kubernetes.io/instance=${cluster_name} -n ${namespace} &&
kubectl delete pod -l app.kubernetes.io/component=tikv,app.kubernetes.io/instance=${cluster_name} -n ${namespace}
```