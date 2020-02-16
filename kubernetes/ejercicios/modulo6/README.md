# Almacenamiento

## Ejemplo 1: PV y PVC (estático)

Creamos el PersistantVolumen:

    kubectl create -f pv.yaml

    kubectl get pv
    NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
    pv1       5Gi        RWX               Recycle             Available         manual                    5s

Creamos el PersistantVolumenClaim:

    kubectl create -f pvc1.yaml

    kubectl get pv          
    NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS   REASON   AGE
    pv1       5Gi         RWX               Recycle             Bound       default/pvc1  manual                    66s
    
    kubectl get pvc
    NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    pvc1   Bound       pv1   5Gi          RWX            manual           31s

Escribimos un fichero index.html en el directorio correspondiente al volumen:

    minikube ssh                           
    $ sudo sh -c "echo 'Hello from Kubernetes storage' > /data/pv1/index.html"

Creamos un pod con el volumen

    kubectl create -f pod.yaml
    kubectl get pod
    kubectl describe pod task-pv-pod

Accedemos el pod, instalamos curl y probamos a acceder al servidor web

    kubectl exec -it task-pv-pod -- /bin/bash

    root@task-pv-pod:/# apt-get update
    root@task-pv-pod:/# apt-get install curl
    root@task-pv-pod:/# curl localhost
    Hello from Kubernetes storage


## Ejemplo 2: Gestión dinámica de volúmenes

En minikube, tenemos un provisionador de almacenamiento local del tipo hostPath. 

    kubectl get storageclass
    NAME                 PROVISIONER                 AGE
    standard (default)   k8s.io/minikube-hostpath   2d23h

Por la tanto si creamos un PersistantVolumenClaim, se creará de forma dinámica un PV:

    kubectl create -f pvc2.yaml

    kubectl get pv
    NAME                                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS   
    pvc-fba7b6e9-817e-11e9-9dd3-080027b9a41d   1Gi        RWX            Delete           Bound    default/pvc2   standard            

    kubectl get pvc
    NAME   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    pvc2   Bound    pvc-fba7b6e9-817e-11e9-9dd3-080027b9a41d   1Gi           RWX            standard       15s


    kubectl delete pvc pvc2
    kubectl get pv


## Ejemplo 3: WordPress con almacenamiento persistente

    kubectl create -f wordpress-pv.yaml
    kubectl create -f wordpress-ns.yaml

    kubectl create -f wordpress-pvc.yaml 
    kubectl create -f mariadb-pvc.yaml 
    kubectl get pv,pvc -n wordpress    

    kubectl apply -f mariadb-srv.yaml
    kubectl apply -f wordpress-srv.yaml
    kubectl apply -f wordpress-ingress.yaml
    kubectl apply -f mariadb-secret.yaml
    kubectl apply -f mariadb-deployment.yaml
    kubectl apply -f wordpress-deployment.yaml

    kubectl get all -n wordpress

