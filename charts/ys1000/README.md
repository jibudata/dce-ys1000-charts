# 公司介绍
骥步科技是一家云原生数据存储和灾备服务提供商，专注于企业级数据管理和数据保护领域，以容器和Kubernetes为基础的云原生技术是云计算的颠覆性变革，带来了以开发者和应用为中心的全新生态，提供了高度自动化的基础设施服务。当前主要提供一款云原生灾备产品—— YS1000。

---
# 产品介绍
YS1000企业版是骥步科技自主设计和研发的国内首个Kubernetes备份容灾商业解决方案，它提供了面向容器应用的本地和异地备份与恢复服务，多集群的备份和恢复任务管理，多云环境中的应用迁移及容灾保护服务。YS1000能够快速帮忙企业搭建容器环境的备份容灾能力，保护应用数据安全，为企业业务正常运行保驾护航。

---
# 部署步骤
此软件(Helm chart) 使用 **Helm** 包管理工具在道客的云原生应用平台中安装和部署银数多云数据管理软件。

## 先决条件

- Kuberentes 版本支持 >= Kubernetes 1.18
- Helm 版本支持
- 在线安装请确保K8S集群节点可以访问和拉取容器镜像 (container images)
- S3 (AWS S3兼容) 对象存储（可采用minio用于私有云部署，按照yaml详见文字末尾）

## 在线安装

1. 使用以下命令添加 **Helm** 软件仓库:

   ```bash
   $ helm repo add qiming https://jibutech.github.io/helm-charts/
   ```

   添加完成之后，您可以通过执行命令 `helm search repo qiming` 来查看可选安装的软件版本，例如:

   ```bash
   [root@test-master ~]# helm search repo qiming
   NAME                  	CHART VERSION	APP VERSION	DESCRIPTION
   qiming/qiming-operator	2.0.0        	2.0.0      	A Helm chart for yinhestor data management plat...
   ```

> 注意： 如果版本有更新，安装qiming最新版

2. 您可以使用以下两种方法进行安装:

   **注意-1**: 为确保安装成功，请设置必需的的配置参数， 具体信息请参考安装手册配置章节 `<br>`
   **注意-2**: 需要在安装本软件之前准备好 S3 (AWS S3兼容) 对象存储环境，下文基于本地安装的 [minio](https://min.io/) 为例进行说明

   1. 通过命令行方式安装:

      使用**Helm**命令行参数 `--set key=value[,key=value] `来指定必要的配置参数，例如:

      ```bash
      helm install qiming/qiming-operator --namespace qiming-migration \ 
          --create-namespace --generate-name --set service.type=NodePort \
          --set s3Config.provider=aws --set s3Config.name=minio \
          --set s3Config.accessKey=minio --set s3Config.secretKey=passw0rd \
          --set s3Config.bucket=test --set s3Config.s3Url=http://172.16.0.10:30170
      ...
      ### 这里配置minio的参数，用于存储备份数据，也可以在安装之后在管理页面进行调整

      NAME: qiming-operator-1618982398


      LAST DEPLOYED: Wed Apr 21 13:19:58 2021
      NAMESPACE: qiming-migration
      STATUS: deployed
      REVISION: 1
      NOTES:
      1. Check the application status Ready by running these commands:
        NOTE: It may take a few minutes to pull docker images.
              You can watch the status of by running `kubectl --namespace qiming-migration get migconfigs.migration.yinhestor.com -w`
        kubectl --namespace qiming-migration get migconfigs.migration.yinhestor.com

      2. After status is ready, get the application URL by running these commands:
        export NODE_PORT=$(kubectl get --namespace qiming-migration -o jsonpath="{.spec.ports[0].nodePort}" services ui-service-default )
        export NODE_IP=$(kubectl get nodes --namespace qiming-migration -o jsonpath="{.items[0].status.addresses[0].address}")
        echo http://$NODE_IP:$NODE_PORT

      3. Login web UI with the token by running these commands:
        export SECRET=$(kubectl -n qiming-migration get secret | (grep qiming-operator |grep -v helm || echo "$_") | awk '{print $1}')
        export TOKEN=$(kubectl -n qiming-migration describe secrets $SECRET |grep token: | awk '{print $2}')
        echo $TOKEN
      ```
   2. 通过 **YAML** 文件指定参数进行安装

      在 **values.yaml** 配置文件中设置或者修改必要的配置参数。

      1. 下载 values.yaml 模板配置文件
   3. 修改配置文件中的配置参数

      3. 通过 `-f values.yaml` 来指定配置文件进行安装， 如下示例：

      ```bash
      # step 1: generate default values.yaml
      helm inspect values  qiming/qiming-operator > values.yaml

      # step 2: fill required arguments in values.yaml
      vim values.yaml

      # step 3: install by specifying the values.yaml
      helm install qiming/qiming-operator --namespace qiming-migration -f values.yaml --generate-name
      ```
3. 获取已安装的软件的运行状态以及访问信息

   1. 使用上述安装结束后 `NOTES` 中的第一条命令来查询软件的运行状态，使用可选 `-w` 参数观察软件初始化过程更新。

      **注意**：软件在初次安装时，需要一段时间下载容器镜像到K8S节点上，具体时间取决于镜像拉取速度。

      当 `PHASE` 状态变成 `Ready`，表明软件初始化完成。

      如果变成 `Error` ，则说明初始化过程失败，需要查找错误原因。

      ```bash
      [root@remote-dev ~]# kubectl --namespace qiming-migration get migconfigs.migration.yinhestor.com -w
      NAME        AGE     PHASE   CREATED AT             VERSION
      migconfig   2m20s   Ready   2021-04-21T05:19:58Z   v0.2.1
      ```
   2. 使用上述安装结束后 `NOTES` 中的第二条和第三条命令获取程序访问地址(Web URL) 以及登录所需的认证令(token)

      **注意**：软件通过K8S Service 来暴露对外访问接口；Web URL 基于K8S 配置以及安装中指定的参数不同， 暴露出的访问地址不同，支持类型包括: `kubectl proxy`， `ingress` 和  `nodeport`，下文以 `nodeport` 安装方式为例:

      ```
      ### 复制下面代码，直接在linux的master上执行，获取软件的登录方式和账号密码
      cat > ys1000.sh << EOF
      #!/bin/bash
      echo "#############################################################"
      echo "ys1000的访问地址是："
        export NODE_PORT=$(kubectl get --namespace qiming-migration -o jsonpath="{.spec.ports[0].nodePort}" services ui-service-default )
        export NODE_IP=$(kubectl get nodes --namespace qiming-migration -o jsonpath="{.items[0].status.addresses[0].address}")
        echo http://$NODE_IP:$NODE_PORT
      echo "#############################################################"
      echo "##################     集群账号登录方式     ###################"
      echo "ys1000的集群账号令牌是："
        export SECRET=$(kubectl -n qiming-migration get secret | (grep qiming-operator |grep -v helm || echo "$_") | awk '{print $1}')
        export TOKEN=$(kubectl -n qiming-migration describe secrets $SECRET |grep token: | awk '{print $2}')
        echo $TOKEN

      echo "#############################################################"
      echo "####################    密码登录方式    #####################"
      echo "默认的用户名是admin，默认的密码是：passw0rd"
      echo "#############################################################"
      EOF
      bash ys1000.sh
      ```
   3. 使用命令 `helm list -n <NAMESPACE> ` 来显示当前安装的软件信息，例如：

      ```bash
      [root@remote-dev ~]# helm list -n qiming-migration
      NAME                      	NAMESPACE       	REVISION	UPDATED                              	STATUS  	CHART                	APP VERSION
      qiming-operator-1618982398	qiming-migration	1       	2021-04-21 13:19:58.7127374 +0800 CST	deployed	qiming-operator-0.2.1	0.2.1
      ```

## 升级

1. 使用命令 `helm repo update` 更新可用的软件版本, 并可以通过 `helm search repo qiming` 来查看更新后的软件版本列表，例如：

   ```bash
   [root@ ~]# helm search repo qiming
   NAME                  	CHART VERSION	APP VERSION	DESCRIPTION
   qiming/qiming-operator	1.0.1        	1.0.1      	A Helm chart for yinhestor data management plat...
   [root@ ~]# helm search repo qiming --versions
   NAME                  	CHART VERSION	APP VERSION	DESCRIPTION
   qiming/qiming-operator	1.0.1        	1.0.1      	A Helm chart for yinhestor data management plat...
   qiming/qiming-operator	1.0.0        	1.0.0      	A Helm chart for yinhestor data management plat...
   qiming/qiming-operator	0.2.1        	0.2.1      	A Helm chart for yinhestor data management plat...
   helm search repo qiming --versions
   ```
2. 使用命令  `helm upgrade` 进行软件升级，通过参数 `--version=<CHART VERSION>`  指定升级版本， 可选参数 `--reuse-values` 用来保留之前安装的配置参数

   **注意**：如果需要在升级过程中修改或者增加部分参数，可以附加参数 `--set key=value[,key=value] ` 来完成。

   例如：

   ```bash
   [root@remote-dev ~]helm upgrade qiming-operator-1618982398 qiming/qiming-operator --namespace qiming-migration --reuse-values --version=0.2.2
   ```

## 卸载

1. 卸载已安装的 Helm chart

   指定当前已安装的软件名 `release name` 和 软件所在的命名空间 `namespace`

   ```bash
   [root@remote-dev ~]# helm list -n qiming-migration
   NAME                      	NAMESPACE       	REVISION	UPDATED                                	STATUS  	CHART                	APP VERSION
   qiming-operator-1618982398	qiming-migration	4       	2021-04-21 13:41:27.365865385 +0800 CST	deployed	qiming-operator-0.2.1	0.2.1

   [root@remote-dev ~]# helm uninstall qiming-operator-1618982398 -n qiming-migration
   release "qiming-operator-1618982398" uninstalled
   ```

   **注意**:  `velero`  组件和资源对象默认仍然保留在命名空间中, 已防止数据丢失。

   如果您确定需要删除 `velero` 和相关应用备份数据记录，可以通过下列命令在每个Kubernetes 集群上分别运行，进行清除操作：

   ```bash
   kubectl delete ns qiming-migration

   kubectl delete clusterrolebindings.rbac.authorization.k8s.io velero-qiming-migration

   kubectl delete crds -l component=velero
   ```

## minio部署（非生产）

注意修改文件中的存储类名称，或者自己创建存储卷

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: minio

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: minio
  name: minio
  labels:
    component: minio
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      component: minio
  template:
    metadata:
      labels:
        component: minio
    spec:
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: miniopvc
      containers:
      - name: minio
        image: minio/minio:RELEASE.2020-05-01T22-19-14Z
        imagePullPolicy: IfNotPresent
        args:
          - server
          - /storage
        env:
        - name: MINIO_ACCESS_KEY
          value: "minio"
        - name: MINIO_SECRET_KEY
          value: "minio123"
        ports:
        - containerPort: 9000
        volumeMounts:
        - name: storage
          mountPath: "/storage"
---
apiVersion: v1
kind: Service
metadata:
  namespace: minio
  name: minio
  labels:
    component: minio
spec:
  type: NodePort
  ports:
    - nodePort: 30165
      port: 9000
      targetPort: 9000
      protocol: TCP
  selector:
    component: minio
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: miniopvc
  namespace: minio
spec:
  storageClassName: daocloud  # 请替换成本集群的存储类，使用命令 kubectl get sc 获取
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

# 联系方式

<p>请联系道客生态团队获取企业级支持: <a href="mailto:peg-pem@daocloud.io">peg-pem@daocloud.io</a></p>
