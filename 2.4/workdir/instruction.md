

##  Создание пользователя или группу ssl

``openssl genrsa -out test.key 2048``

`` openssl req -new -key test.key -out test.csr -subj "/CN=test/O=ops"``
 
 где:
 - CN - это имя
 - O - это группа

### create user cert
Kuber:

``openssl x509 -req -in test.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out test.crt -days 5000``

minikube:

``openssl x509 -req -in test.csr -CA .minikube/ca.crt -CAkey .minikube/ca.key  -CAcreateserial -out test.crt -days 5000``

microk8s:

``openssl x509 -req -in test.csr -CA /var/snap/microk8s/current/certs/ca.crt -CAkey /var/snap/microk8s/current/certs/ca.key  -CAcreateserial -out test.crt -days 5000``

## Добавить конфиг 

``kubectl config set-credentials a.prisyazhnyuk --client-certificate=a.prisyazhnyuk.crt --client-key=a.prisyazhnyuk.key --embed-certs=true``

kuber:

``kubectl config set-context a.prisyazhnyuk --cluster=kubernetes --user=a.prisyazhnyuk``

minikube:

``kubectl config set-context test --cluster=minikube --user=test``

Microk8s:

``kubectl config set-context test --cluster=microk8s --user=test``

``kubectl config set-context a.prisyazhnyuk --cluster=microk8s-cluster --user=a.prisyazhnyuk``


``kubectl config use-context test``

## Создаем Role

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: test
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```
## Прикрепляем роль к пользователю/группе RoleBinding

rolebinding.yaml:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: test
subjects:
- kind: Group #User
  name: ops   #test
  apiGroup: rbac.authorization.k8s.io
roleRef
  kind: Role
  name: test
  apiGroup: rbac.authorization.k8s.io

```


# Token

``kubectl get sa``

```kubectl create sa netology # создает service account```

``kubectl apply -f secret-sa-token.yaml`` - создаем токен
```
apiVersion: v1
kind: Secret
metadata:
  name: netology-token
  annotations:
    kubernetes.io/service-account.name: "netology"
type: kubernetes.io/service-account-token
```
``kubectl get sa``

``kubectl describe sa netology``

Далее выводим секрет токена:

``kubectl describe secret netology-token``

``export TOKEN=""``

``curl --cacert .minikube/ca.crt --header  "Authorization: Bearer ${TOKEN}" -X GET https://192.168.27.83:8334/api``

## Просмотр конфига и использования контекста
``kubectl config get-context``

``kubectl config use-context test``

