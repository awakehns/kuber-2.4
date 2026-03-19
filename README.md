# Домашнее задание к занятию `«Управление доступом»` - `Демин Герман`

## Задание 1

```
openssl genrsa -out user.key 2048

openssl req -new -key user.key -out user.csr -subj "/CN=dev-user"

nano csr.yaml

echo "$(base64 user.csr | tr -d '\n')" > base64
```

`cat base64`

Вставил это в csr.yaml в секцию request

[csr.yaml](csr.yaml)

`nano role.yaml`

[role.yaml](role.yaml)

`nano rolebinding.yaml`

[rolebinding.yaml](rolebinding.yaml)

`kubectl apply -f .`

```
kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}'

kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 -d > ca.crt

kubectl config set-cluster dev-cluster \
  --server=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}') \
  --certificate-authority=$HOME/.minikube/ca.crt \
  --embed-certs=true \
  --kubeconfig=dev-user.kubeconfig

kubectl certificate approve dev-user-csr
  
kubectl config set-credentials dev-user \
  --client-certificate=$HOME/.minikube/ca.crt \
  --client-key=user.key \
  --embed-certs=true \
  --kubeconfig=dev-user.kubeconfig

kubectl config set-context dev-user-context \
  --cluster=dev-cluster \
  --user=dev-user \
  --namespace=default \
  --kubeconfig=dev-user.kubeconfig
  
kubectl config use-context dev-user-context \
  --kubeconfig=dev-user.kubeconfig

kubectl get role,rolebinding

kubectl auth can-i get pods --as=dev-user

kubectl auth can-i get pods/log --as=dev-user

kubectl auth can-i delete pods --as=dev-user
```

![kube-1](/img/kube-1.png)
