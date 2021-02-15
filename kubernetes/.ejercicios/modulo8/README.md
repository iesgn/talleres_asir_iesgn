# Administración básica

## Namespaces

    kubectl get namespaces
    NAME          STATUS    AGE
    default       Active    1d
    kube-public   Active    1d
    kube-system   Active    1d

Para crear un nuevo namespace:

    kubectl create ns proyecto1
    kubectl describe ns proyecto1

Para crear un recurso en un namespace:

    kubectl create deployment nginx --image=nginx -n proyecto1
    kubectl expose deployment nginx --port=80 --type=NodePort -n proyecto1
    kubectl get deploy -n proyecto1
    kubectl get deploy --all-namespaces

Al borrar un namespaces se borran los recursos creados (los volúmenes no se asocian a namespaces):

    kubectl delete ns proyecto1

## Usuarios

    kubectl create namespace proyecto1

Creamos la clave privada (en ~/.certs):

    openssl genrsa -out usuario1.key 2048

Creamos la petición de certificado (CSR):

    openssl req -new -key usuario1.key -out usuario1.csr -subj "/CN=usuario1/O=desarrollo"

Mandamos el CSR al administrador

Crea el certificado a partir de CSR firmándolo con el CA:

    openssl x509 -req -in usuario1.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out usuario1.crt -days 500

**Configuración**

Suponemos que el usuario se está conectando a un cluster que ya tiene configurado en `~/.kube/config`

Añadimos las credenciales del nuevo usuario:

    kubectl config set-credentials usuario1 --client-certificate=/home/debian/.certs/usuario1.crt  --client-key=/home/debian/.certs/usuario1.key

Añadimos el nuevo contexto:

    kubectl config set-context usuario1-context --cluster=minikube --namespace=proyecto1 --user=usuario1

**Acceso**

Mostramos los contextos definidos en el fichero de configuración:

    kubectl config get-contexts
    CURRENT   NAME               CLUSTER    AUTHINFO   NAMESPACE
              usuario1-context   minikube   usuario1   proyecto1
    *         minikube             minikube   minikube  

Elegimos el nuevo contexto:

    kubectl config use-context usuario1-context

    kubectl get pods
    Error from server (Forbidden): pods is forbidden: User "usuario1" cannot list resource "pods" in API group "" in the namespace "proyecto1"

## Ejercicio 1: Role y ReoleBinding

Vamos a crear los roles como administrador:

    kubectl config use-context minikube

    kubectl create -f role-deployment-manager.yaml
    kubectl get role -n proyecto1
    kubectl describe role -n proyecto1

    kubectl create -f rolebinding-deployment-manager.yaml
    kubectl describe rolebinding -n proyecto1

Ahora podemos acceder como usuario1 y comprobamos si podemos trabajar con los recursos que hemos autorizado:

    kubectl config use-context usuario1-context
    kubectl get pods
    No resources found.

## ServiceAccount

Veamos un ejemplo: instalamos Prometheus con helm:

    helm install \
        --namespace=monitoring \
        --name=prometheus \
        --version=7.0.0 \
        stable/prometheus


    kubectl get pod -n monitoring
    NAME                                             READY   STATUS    RESTARTS   AGE
    ...
    prometheus-server-559fbf685c-cgmvm               2/2     Running   2          19s
    
    kubectl get serviceaccount -n monitoring
    NAME                            SECRETS   AGE
    default                         1         19s
    ...
    prometheus-server               1         19s

    kubectl describe serviceaccount prometheus-server -n monitoring
    ...
    Tokens:              prometheus-server-token-xx2pq

    kubectl get secrets -n monitoring                              
    NAME                                        TYPE                                  DATA   AGE
    ...
    prometheus-server-token-xx2pq               kubernetes.io/service-account-token   3      19s

    kubectl describe secret prometheus-server-token-xx2pq -n monitoring

    kubectl describe pod prometheus-server-559fbf685c-cgmvm -n monitoring
    ...
        Mounts:
          /data from storage-volume (rw)
          /var/run/secrets/kubernetes.io/serviceaccount from prometheus-server-token-xx2pq (ro)
    ...


## Ejemplo 2: Quotas y Limits

    kubectl create -f quota.yaml

    kubctl get resourcequota
    kubectl describe resourcequota mem-cpu-demo
    ...
    Resource         Used  Hard
    --------         ----  ----
    limits.cpu       0     2
    limits.memory    0     2Gi
    requests.cpu     0     1
    requests.memory  0     1Gi

    kubectl describe ns default

    kubectl create -f pod1.yaml

    kubectl describe resourcequota mem-cpu-demo
    ...
    Resource         Used   Hard
    --------         ----   ----
    limits.cpu       800m   2
    limits.memory    800Mi  2Gi
    requests.cpu     400m   1
    requests.memory  600Mi  1Gi

    kubectl create -f pod2.yaml
    Error from server (Forbidden): error when creating "pod2.yaml": pods "quota-mem-cpu-demo-2" is forbidden: exceeded quota: mem-cpu-demo, requested: requests.memory=700Mi, used: requests.memory=600Mi, limited: requests.memory=1Gi


