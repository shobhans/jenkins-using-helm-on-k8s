# Install Jenkins on Kubernetes using Helm chart

### Install helm

https://helm.sh/docs/intro/install/

### Create Namespace, ServiceAccount, ClusterRole, ClusterRoleBinding, StorageClass, PersistentVolume, PersistentVolumeClaim

- local StorageClass is used on this example.
- local PersistentVolume (path: /jenkinsdata on worker node: worker1) is used on this example.
- update worker node name ('worker1' in this example) on nodeAffinity section of PersistentVolume on jenkins-pv.yaml file.

```bash
ssh root@worker1
mkdir /jenkinsdata
```

```bash
kubectl apply -f jenkins-ns-sa-clusterrole.yaml
kubectl apply -f jenkins-pv.yaml
```

### Install Jenkins via helm

- update jenkins-values.yaml using the following key value pairs. Values should be updated as required.
- NodePort service type is used on this example.

```yaml
nodePort: 30081
###
persistence:
  enabled: true
  existingClaim: jenkins-pv-claim
###
storageClass: local-storage
###
serviceAccount:
  create: false
  name: jenkins
  annotations: {}
```

```bash
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
helm search repo jenkinsci
helm install jenkins -n jenkins -f jenkins-values.yaml jenkinsci/jenkins
```

### Access Jenkins console

- get 'admin' user password

```bash
jsonpath="{.data.jenkins-admin-password}"
secret=$(kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath)
echo $(echo $secret | base64 --decode)
```

- get login url

```bash
jsonpath="{.spec.ports[0].nodePort}"
NODE_PORT=$(kubectl get -n jenkins -o jsonpath=$jsonpath services jenkins)
jsonpath="{.items[0].status.addresses[0].address}"
NODE_IP=$(kubectl get nodes -n jenkins -o jsonpath=$jsonpath)
echo http://$NODE_IP:$NODE_PORT/login
```

### Links

- local PersistentVolume https://kubernetes.io/docs/concepts/storage/volumes/#local
- Install Jenkins on K8S using helm - https://www.jenkins.io/doc/book/installing/kubernetes/#install-jenkins-with-helm-v3
