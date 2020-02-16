# Otros tipos de despliegues

## Ejemplo 1: StatefulSet

Vamos a crear los distintos objetos de la API:

    kubectl create -f service.yaml

Creación ordenada de pods: En un terminal observamos la creación de pods y en otro terminal creamos los pods

    watch kubectl get pod
    kubectl create -f statefulset.yaml

Comprobamos la identidad de red estable: Vemos los hostnames y los nombres DNS asociados:

    for i in 0 1; do kubectl exec web-$i -- sh -c 'hostname'; done
    web-0
    web-1

    kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
    / # nslookup web-0.nginx
    ...
    Address 1: 172.17.0.4 web-0.nginx.default.svc.cluster.local
    / # nslookup web-1.nginx
    ...
    Address 1: 172.17.0.5 web-1.nginx.default.svc.cluster.local

Creación  ordenada de pods: En un terminal observamos la creación de pods y en otro terminal eliminamos los pods

    watch kubectl get pod
    kubectl delete pod -l app=nginx

Comprobamos la identidad de red estable: Vemos los hostnames y los nombres DNS asociados (Las IP pueden cambiar):

    for i in 0 1; do kubectl exec web-$i -- sh -c 'hostname'; done
    web-0
    web-1

    kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
    / # nslookup web-0.nginx
    ...
    Address 1: 172.17.0.4 web-0.nginx.default.svc.cluster.local
    / # nslookup web-1.nginx
    ...
    Address 1: 172.17.0.5 web-1.nginx.default.svc.cluster.local


## Ejemplo 2: DaemontSet

    kubectl get nodes                 
    NAME    STATUS   ROLES    AGE   VERSION
    k3s-1   Ready    <none>   17d   v1.14.1-k3s.4
    k3s-2   Ready    <none>   17d   v1.14.1-k3s.4
    k3s-3   Ready    <none>   17d   v1.14.1-k3s.4

    kubectl create -f /tmp/ds.yaml

    kubectl get pods -o wide      
    NAME            READY   STATUS    RESTARTS   AGE   IP       NODE
    logging-5v4dh   1/1     Running   0          9s    10.42.2.26   k3s-2
    logging-gqfbb   1/1     Running   0          9s    10.42.1.55   k3s-3
    logging-wbdjj   1/1     Running   0          9s    10.42.0.25   k3s-1


Podemos seleccionar los nodos en los que queremos que se ejecuten los pod por medio de un selector.

    kubectl create -f /tmp/ds2.yaml

    kubectl get pods -o wide      
    No resources found.

    kubectl get ds          
    NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR      AGE
    logging   0         0         0       0            0           app=logging-node   17s

    kubectl label node k3s-3 app=logging-node --overwrite

    kubectl get ds                                       
    NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR      AGE
    logging   1         1         0       1            0           app=logging-node   41s

    kubectl get pods -o wide                             
    NAME            READY   STATUS    RESTARTS   AGE   IP           NODE    NOMINATED NODE   READINESS GATES
    logging-556r9   1/1     Running   0          7s    10.42.1.56   k3s-3   <none>           <none>


## Horizontal Pod AutoScaler

Veamos un ejemplo: creamos un despliegue de una aplicación php y modificamos lo que va a reservar del CPU el pod (0,2 cores de CPU):

    kubectl create deploy php-apache --image=k8s.gcr.io/hpa-example
    kubectl expose deploy php-apache --port=80 --type=NodePort
    kubectl set resources deploy php-apache --requests=cpu=200m

Creamos el recurso hpa, indicando el mínimo y máximos de pods que va a terner el despliegue, y el límite de uso de CPU que va a tener en cuenta para crear nuevos pods:

    kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=5

Vamos a hacer una prueba de estrés a nuestra aplicación y observamos cómo se comporta:

    $ while true; do wget -q -O- http://192.168.99.100:30372/; done

    kubectl get pod -w
    kubectl get hpa -w

## helm

Inicializamos helm

    helm init

Actualizamos el repositorio

    helm repo update

Buscamos un chart

    helm search wordpress

Instalamos el chart

    helm install --set serviceType=NodePort --name wp-k8s stable/wordpress

Listamos las aplicaciones instaladas

    helm ls

Borramos la aplicación

    helm delete wp-k8s

Vemos los recursos creados

    helm status wp-k8s


