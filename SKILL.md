---
name: k8s-fta-skill
description: 基于FTA故障树分析法的Kubernetes问题定位工具。当用户遇到k8s集群问题、Pod运行异常、服务访问失败、RBAC权限问题、DNS解析失败、OOMKilled、健康检查失败、网络策略限制、存储挂载问题、HPA扩展问题、API Server连接问题等情况时，使用此技能提供系统化的故障排查流程和解决方案。同时支持k3s轻量级Kubernetes发行版的故障排查。Supports both Chinese and English troubleshooting for Kubernetes cluster issues including Pod failures, service access problems, RBAC issues, DNS resolution failures, OOMKilled, health check failures, network policy restrictions, storage mount issues, HPA scaling problems, and API Server connectivity issues. Also supports k3s lightweight Kubernetes distribution.
compatibility:
  tools: []
---

# Kubernetes FTA故障树分析技能 / Kubernetes FTA Fault Tree Analysis Skill

## 技能简介 / Skill Introduction

本技能基于FTA（故障树分析）方法，提供系统化的Kubernetes集群故障排查流程。通过逐步分析，帮助用户快速定位和解决k8s环境中的各种问题，包括Pod运行异常、服务访问失败、网络通信问题、RBAC权限问题、DNS解析失败、OOMKilled、健康检查失败、网络策略限制、存储挂载问题、HPA扩展问题、API Server连接问题等。

This skill provides a systematic Kubernetes cluster troubleshooting process based on FTA (Fault Tree Analysis) methodology. Through step-by-step analysis, it helps users quickly locate and resolve various issues in k8s environments, including Pod failures, service access problems, network communication issues, RBAC permission issues, DNS resolution failures, OOMKilled, health check failures, network policy restrictions, storage mount issues, HPA scaling problems, API Server connectivity issues, and more.

## 故障排查流程

### 1. 开始排查

首先，执行以下命令检查集群中所有Pod的状态：

```bash
kubectl get pods
```

### 2. Pod状态检查

#### 2.1 是否有Pod处于非Running状态？
- **是**：执行 `kubectl describe pod <pod-name>` 查看详细信息
- **否**：继续下一步

#### 2.2 查看Pod事件和状态
- 检查是否达到了资源配额限制
- 检查是否存在PersistentVolumeClaim问题
- 检查是否存在镜像拉取失败
- 检查是否存在CrashLoopBackOff状态

### 3. 资源配额检查

如果Pod处于Pending状态，检查是否达到了资源配额限制：

```bash
kubectl describe pod <pod-name>
```

### 4. 存储问题检查

如果存在PersistentVolumeClaim问题，检查存储配置：

```bash
kubectl describe pvc <pvc-name>
kubectl describe pv <pv-name>
```

### 5. 镜像问题检查

如果存在镜像拉取失败：
- 检查镜像名称是否正确
- 检查镜像Tag是否存在
- 检查是否需要从私有仓库拉取镜像（认证问题）
- 检查CRI/容器运行时是否正常

### 6. 容器启动问题检查

如果Pod处于CrashLoopBackOff状态：

```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
```

### 7. 网络通信检查

执行端口转发测试Pod内部服务是否正常：

```bash
kubectl port-forward <pod-name> 8080:<pod-port>
```

### 8. Service配置检查

如果服务访问失败，检查Service配置：

```bash
kubectl describe service <service-name>
```

- 检查Service是否正确选择了Pod
- 检查Service的端口配置是否正确
- 执行端口转发测试Service是否正常：
  ```bash
  kubectl port-forward service/<service-name> 8080:<service-port>
  ```

### 9. Ingress配置检查

如果通过Ingress访问失败，检查Ingress配置：

```bash
kubectl describe ingress <ingress-name>
```

- 检查Ingress规则是否正确
- 检查Ingress控制器是否正常运行
- 执行端口转发测试Ingress后端服务是否正常：
  ```bash
  kubectl port-forward <ingress-pod-name> 8080:80
  ```

### 10. 节点状态检查

如果多个Pod出现问题，检查节点状态：

```bash
kubectl get nodes
kubectl describe node <node-name>
```

- 检查节点是否处于NotReady状态
- 检查节点资源使用情况
- 检查kubelet是否正常运行

### 11. OOMKilled问题检查

如果Pod被OOMKilled（内存不足）：

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

- 检查Pod的内存限制设置
- 检查容器的实际内存使用情况
- 调整内存限制或优化应用程序内存使用

### 12. 健康检查失败检查

如果Pod因健康检查失败而重启：

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

- 检查livenessProbe配置
- 检查readinessProbe配置
- 验证健康检查端点是否正常响应
- 调整健康检查参数（超时时间、间隔等）

### 13. RBAC权限问题检查

如果遇到权限拒绝错误：

```bash
kubectl auth can-i <verb> <resource> --as=<user> --namespace=<namespace>
kubectl describe clusterrole <role-name>
kubectl describe clusterrolebinding <binding-name>
```

- 检查ServiceAccount的权限配置
- 检查Role/ClusterRole的权限设置
- 检查RoleBinding/ClusterRoleBinding的绑定关系
- 验证用户或ServiceAccount是否有足够的权限

### 14. DNS解析问题检查

如果Pod无法解析域名：

```bash
kubectl exec -it <pod-name> -- nslookup <domain>
kubectl exec -it <pod-name> -- cat /etc/resolv.conf
kubectl get svc -n kube-system kube-dns
```

- 检查CoreDNS/Kube-DNS是否正常运行
- 检查DNS配置（/etc/resolv.conf）
- 检查网络策略是否阻止DNS访问
- 验证Service的DNS名称是否正确

### 15. 网络策略限制检查

如果Pod无法访问其他服务：

```bash
kubectl get networkpolicies --all-namespaces
kubectl describe networkpolicy <policy-name>
```

- 检查是否有网络策略限制流量
- 验证网络策略的规则配置
- 检查Pod的标签是否匹配网络策略选择器
- 调整网络策略规则以允许必要的流量

### 16. 持久化存储挂载问题检查

如果Pod无法挂载存储卷：

```bash
kubectl describe pod <pod-name>
kubectl get pv
kubectl get pvc
```

- 检查PVC是否处于Bound状态
- 检查PV是否可用
- 验证存储类（StorageClass）配置
- 检查存储驱动是否正常工作

### 17. 容器启动超时检查

如果Pod因启动超时而失败：

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

- 检查应用程序启动时间
- 调整readinessProbe的初始延迟时间
- 优化应用程序启动流程
- 增加terminationGracePeriodSeconds

### 18. 节点污点和容忍度检查

如果Pod无法调度到特定节点：

```bash
kubectl describe node <node-name>
kubectl get pods -o wide
kubectl describe pod <pod-name>
```

- 检查节点的污点（Taints）配置
- 检查Pod的容忍度（Tolerations）配置
- 验证节点选择器（NodeSelector）配置
- 调整污点或容忍度配置

### 19. Pod优先级和抢占检查

如果Pod被抢占或无法调度：

```bash
kubectl describe pod <pod-name>
kubectl get priorityclass
```

- 检查Pod的优先级类（PriorityClass）
- 检查集群中是否有更高优先级的Pod
- 验证资源请求和限制设置
- 调整Pod优先级或资源配置

### 20. HPA/VPA自动扩展问题检查

如果自动扩展不工作：

```bash
kubectl get hpa
kubectl describe hpa <hpa-name>
kubectl get vpa
kubectl describe vpa <vpa-name>
```

- 检查HPA/VPA配置是否正确
- 验证metrics-server是否正常运行
- 检查Pod的资源使用情况
- 调整扩展参数（最小/最大副本数、目标CPU/内存使用率等）

### 21. Pod中断预算问题检查

如果Pod在更新时被意外终止：

```bash
kubectl get pdb
kubectl describe pdb <pdb-name>
```

- 检查PDB配置是否正确
- 验证最小可用Pod数量
- 检查Deployment/StatefulSet的更新策略
- 调整PDB参数以符合业务需求

### 22. 容器运行时问题检查

如果容器无法启动或运行异常：

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
systemctl status docker  # 或 containerd
```

- 检查容器运行时是否正常运行
- 检查容器运行时日志
- 验证容器镜像是否兼容
- 重启容器运行时服务

### 23. 集群证书过期检查

如果遇到证书相关错误：

```bash
kubeadm certs check-expiration
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -dates
```

- 检查集群证书的过期时间
- 检查API Server、Controller Manager等组件的证书
- 更新过期的证书
- 重启相关服务

### 24. API Server连接问题检查

如果无法连接到API Server：

```bash
kubectl cluster-info
kubectl get componentstatuses
kubectl logs -n kube-system kube-apiserver-<node-name>
```

- 检查API Server是否正常运行
- 检查kubeconfig配置
- 验证网络连接和防火墙规则
- 检查API Server日志

### 25. etcd集群问题检查

如果etcd集群出现问题：

```bash
kubectl get pods -n kube-system | grep etcd
kubectl logs -n kube-system etcd-<node-name>
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key endpoint health
```

- 检查etcd Pod状态
- 检查etcd集群健康状态
- 验证etcd数据备份
- 检查etcd存储空间

## 常见问题及解决方案 / Common Issues and Solutions

### 1. Pod处于Pending状态 / Pod Pending Status
- **原因 / Cause**：资源不足、调度问题、存储问题 / Resource shortage, scheduling issues, storage problems
- **解决方案 / Solution**：
  - 检查集群资源使用情况 / Check cluster resource usage
  - 调整Pod资源请求和限制 / Adjust Pod resource requests and limits
  - 检查存储配置 / Check storage configuration

### 2. Pod处于CrashLoopBackOff状态 / Pod CrashLoopBackOff Status
- **原因 / Cause**：应用程序错误、配置错误、依赖问题 / Application errors, configuration errors, dependency issues
- **解决方案 / Solution**：
  - 查看Pod日志 / Check Pod logs
  - 检查应用程序配置 / Check application configuration
  - 检查依赖服务是否正常 / Check if dependent services are running

### 3. Service无法访问 / Service Access Failure
- **原因 / Cause**：Service配置错误、Pod选择器不匹配、网络策略限制 / Service configuration errors, Pod selector mismatch, network policy restrictions
- **解决方案 / Solution**：
  - 检查Service配置 / Check Service configuration
  - 验证Pod标签是否与Service选择器匹配 / Verify Pod labels match Service selector
  - 检查网络策略 / Check network policies

### 4. Ingress无法访问 / Ingress Access Failure
- **原因 / Cause**：Ingress配置错误、Ingress控制器问题、后端服务故障 / Ingress configuration errors, Ingress controller issues, backend service failures
- **解决方案 / Solution**：
  - 检查Ingress配置 / Check Ingress configuration
  - 验证Ingress控制器是否正常运行 / Verify Ingress controller is running
  - 检查后端服务是否正常 / Check if backend services are running

### 5. Pod被OOMKilled / Pod OOMKilled
- **原因 / Cause**：内存不足、内存泄漏、内存限制过低 / Insufficient memory, memory leaks, low memory limits
- **解决方案 / Solution**：
  - 增加Pod内存限制 / Increase Pod memory limits
  - 优化应用程序内存使用 / Optimize application memory usage
  - 检查内存泄漏 / Check for memory leaks

### 6. 健康检查失败 / Health Check Failures
- **原因 / Cause**：健康检查配置不当、应用程序启动慢、健康检查端点异常 / Improper health check configuration, slow application startup, unhealthy health check endpoints
- **解决方案 / Solution**：
  - 调整健康检查参数 / Adjust health check parameters
  - 增加初始延迟时间 / Increase initial delay
  - 修复健康检查端点 / Fix health check endpoints

### 7. RBAC权限问题 / RBAC Permission Issues
- **原因 / Cause**：权限配置错误、角色绑定缺失、ServiceAccount权限不足 / Permission configuration errors, missing role bindings, insufficient ServiceAccount permissions
- **解决方案 / Solution**：
  - 检查并修正Role/ClusterRole配置 / Check and fix Role/ClusterRole configuration
  - 验证RoleBinding/ClusterRoleBinding绑定关系 / Verify RoleBinding/ClusterRoleBinding relationships
  - 为ServiceAccount分配足够的权限 / Grant sufficient permissions to ServiceAccount

### 8. DNS解析问题 / DNS Resolution Issues
- **原因 / Cause**：CoreDNS故障、DNS配置错误、网络策略限制 / CoreDNS failures, DNS configuration errors, network policy restrictions
- **解决方案 / Solution**：
  - 检查CoreDNS Pod状态 / Check CoreDNS Pod status
  - 验证DNS配置 / Verify DNS configuration
  - 检查网络策略 / Check network policies

### 9. 网络策略限制 / Network Policy Restrictions
- **原因 / Cause**：网络策略配置错误、Pod标签不匹配 / Network policy configuration errors, Pod label mismatch
- **解决方案 / Solution**：
  - 检查并调整网络策略规则 / Check and adjust network policy rules
  - 验证Pod标签 / Verify Pod labels
  - 允许必要的网络流量 / Allow necessary network traffic

### 10. 存储挂载问题 / Storage Mount Issues
- **原因 / Cause**：PVC未绑定、存储类配置错误、存储驱动问题 / PVC not bound, storage class configuration errors, storage driver issues
- **解决方案 / Solution**：
  - 检查PVC和PV状态 / Check PVC and PV status
  - 验证存储类配置 / Verify storage class configuration
  - 检查存储驱动 / Check storage driver

### 11. 容器启动超时 / Container Startup Timeout
- **原因 / Cause**：应用程序启动慢、健康检查配置不当 / Slow application startup, improper health check configuration
- **解决方案 / Solution**：
  - 优化应用程序启动流程 / Optimize application startup process
  - 调整健康检查初始延迟 / Adjust health check initial delay
  - 增加terminationGracePeriodSeconds / Increase terminationGracePeriodSeconds

### 12. 节点污点和容忍度问题 / Node Taint and Toleration Issues
- **原因 / Cause**：节点污点配置错误、Pod容忍度不足 / Node taint configuration errors, insufficient Pod tolerations
- **解决方案 / Solution**：
  - 检查节点污点配置 / Check node taint configuration
  - 为Pod添加适当的容忍度 / Add appropriate tolerations to Pod
  - 调整节点选择器 / Adjust node selector

### 13. Pod优先级和抢占问题 / Pod Priority and Preemption Issues
- **原因 / Cause**：优先级配置不当、资源不足 / Improper priority configuration, insufficient resources
- **解决方案 / Solution**：
  - 检查PriorityClass配置 / Check PriorityClass configuration
  - 调整Pod优先级 / Adjust Pod priority
  - 增加集群资源 / Increase cluster resources

### 14. HPA/VPA自动扩展问题 / HPA/VPA Auto-scaling Issues
- **原因 / Cause**：metrics-server故障、配置错误、资源使用率低 / metrics-server failures, configuration errors, low resource utilization
- **解决方案 / Solution**：
  - 检查metrics-server状态 / Check metrics-server status
  - 验证HPA/VPA配置 / Verify HPA/VPA configuration
  - 调整扩展参数 / Adjust scaling parameters

### 15. Pod中断预算问题 / Pod Disruption Budget Issues
- **原因 / Cause**：PDB配置不当、更新策略错误 / Improper PDB configuration, incorrect update strategy
- **解决方案 / Solution**：
  - 检查PDB配置 / Check PDB configuration
  - 调整最小可用Pod数量 / Adjust minimum available Pods
  - 修改更新策略 / Modify update strategy

### 16. 容器运行时问题 / Container Runtime Issues
- **原因 / Cause**：容器运行时故障、镜像不兼容 / Container runtime failures, incompatible images
- **解决方案 / Solution**：
  - 检查容器运行时状态 / Check container runtime status
  - 重启容器运行时服务 / Restart container runtime service
  - 验证镜像兼容性 / Verify image compatibility

### 17. 集群证书过期 / Cluster Certificate Expiration
- **原因 / Cause**：证书过期、证书配置错误 / Certificate expiration, certificate configuration errors
- **解决方案 / Solution**：
  - 检查证书过期时间 / Check certificate expiration time
  - 更新过期证书 / Update expired certificates
  - 重启相关服务 / Restart related services

### 18. API Server连接问题 / API Server Connectivity Issues
- **原因 / Cause**：API Server故障、网络连接问题、kubeconfig配置错误 / API Server failures, network connectivity issues, kubeconfig configuration errors
- **解决方案 / Solution**：
  - 检查API Server状态 / Check API Server status
  - 验证网络连接 / Verify network connectivity
  - 检查kubeconfig配置 / Check kubeconfig configuration

### 19. etcd集群问题 / etcd Cluster Issues
- **原因 / Cause**：etcd Pod故障、存储空间不足、网络问题 / etcd Pod failures, insufficient storage space, network issues
- **解决方案 / Solution**：
  - 检查etcd Pod状态 / Check etcd Pod status
  - 清理etcd存储空间 / Clean up etcd storage space
  - 验证etcd集群健康状态 / Verify etcd cluster health status

## 工具使用方法

1. 用户描述k8s集群中遇到的问题
2. 技能根据FTA故障树分析流程，逐步引导用户进行排查
3. 针对每个排查步骤，提供具体的命令和检查要点
4. 根据用户反馈的结果，继续引导下一步排查
5. 最终定位问题并提供解决方案

## 输出格式

当用户遇到k8s问题时，技能应提供以下格式的输出：

### 问题分析
- 问题现象描述
- 可能的原因分析

### 排查步骤
1. **步骤1**：执行命令和检查要点
2. **步骤2**：执行命令和检查要点
3. **步骤3**：执行命令和检查要点

### 解决方案
- 根据排查结果提供具体的解决方案
- 提供预防措施和最佳实践

## 示例

### 示例1：Pod处于CrashLoopBackOff状态

**用户输入**：我的Pod一直处于CrashLoopBackOff状态，如何排查？

**输出**：

### 问题分析
- Pod处于CrashLoopBackOff状态，说明容器启动后立即崩溃
- 可能的原因：应用程序错误、配置错误、依赖问题

### 排查步骤
1. **查看Pod日志**：
   ```bash
   kubectl logs <pod-name>
   ```
   检查是否有错误信息

2. **查看之前的日志**：
   ```bash
   kubectl logs <pod-name> --previous
   ```
   检查崩溃前的错误信息

3. **查看Pod详细信息**：
   ```bash
   kubectl describe pod <pod-name>
   ```
   检查事件和状态信息

### 解决方案
- 根据日志中的错误信息修复应用程序问题
- 检查并修正Pod配置
- 确保依赖服务正常运行

### 示例2：Service无法访问

**用户输入**：我的Service创建成功但无法访问，如何排查？

**输出**：

### 问题分析
- Service无法访问，可能是配置问题或后端Pod问题
- 可能的原因：Service配置错误、Pod选择器不匹配、网络策略限制

### 排查步骤
1. **检查Service配置**：
   ```bash
   kubectl describe service <service-name>
   ```
   检查服务类型、端口配置等

2. **检查后端Pod状态**：
   ```bash
   kubectl get pods -l <service-selector>
   ```
   确保Pod正常运行且与Service选择器匹配

3. **测试端口转发**：
   ```bash
   kubectl port-forward service/<service-name> 8080:<service-port>
   ```
   测试Service是否可以正常访问

### 解决方案
- 修正Service配置错误
- 确保Pod标签与Service选择器匹配
- 检查并调整网络策略

## 总结

本技能通过FTA故障树分析方法，提供了系统化的Kubernetes故障排查流程，帮助用户快速定位和解决各种k8s集群问题。使用此技能时，用户需要提供具体的问题描述和执行命令后的结果，以便技能能够准确引导排查过程并提供有效的解决方案。