# Comunicación entre servicios y acceso desde el exterior

## Ejemplo 1: Services

**Service ClusterIP**

    kubectl create -f deployment.yaml

    kubectl create -f service_ci.yaml

También podríamos haber creado el servicio sin usar el fichero yaml, de la siguiente manera:

    kubectl expose deployment/nginx --port=80 --type=ClusterIP

Podemos ver el servicio que hemos creado:

    kubectl get svc

Puede ser bueno acceder desde exterior, por ejemplo en la fase de desarrollo de una aplicación para probarla:

    kubectl proxy

Y accedemos a la URL:

    http://localhost:8001/api/v1/namespaces/<NAMESPACE>/services/<SERVICE NAME>:<PORT NAME>/proxy/

**Service NodePort**

    kubectl create -f service_np.yaml

También podríamos haber creado el servicio sin usar el fichero yaml, de la siguiente manera:

    kubectl expose deployment/nginx --port=80 --type=NodePort

Podemos ver el servicio que hemos creado:

    kubectl get svc

Desde el exterior accedemos a:

    http://<IP_MASTER>:<PUERTO_ASIGNADO>

## Ejemplo 2: DNS

    kubectl create -f busybox.yaml
    kubectl exec -it busybox -- nslookup nginx
    kubectl exec -it busybox -- wget http://nginx

Comprobando el servidor DNS:

    kubectl get pods --namespace=kube-system -o wide
    kubectl get services --namespace=kube-system
    kubectl exec -it busybox -- cat /etc/resolv.conf

## Balanceo de carga

Hemos creado una imagen docker que nos permite crear un contenedor con una aplicación PHP que muestra el nombre del servidor donde se ejecuta, el fichero index.php:

    <?php echo "Servidor:"; echo gethostname();echo "\n"; ?>

Si tenemos varios pod de esta aplicación, el objeto Service balancea la carga entre ellos:

    kubectl create deployment pagweb --image=josedom24/infophp:v1
    kubectl expose deploy pagweb --port=80 --type=NodePort
    kubectl scale deploy pagweb --replicas=3

Al acceder hay que indicar el puerto asignado al servicio:

    for i in `seq 1 100`; do curl http://192.168.99.100:32376; done
    Servidor:pagweb-84f6d54fb7-56zj6
    Servidor:pagweb-84f6d54fb7-mdvfn
    Servidor:pagweb-84f6d54fb7-bhz4p

## Ejemplo 3: guestbook (parte 2)

    kubectl create -f frontend-deployment.yaml
    kubectl create -f redis-master-deployment.yaml
    kubectl create -f redis-slave-deployment.yaml
    kubectl create -f frontend-srv.yam
    kubectl create -f redis-master-srv.yaml
    kubectl create -f redis-slave-srv.yaml

## Ejemplo 4: Despliegue canary

Creamos el servicio y el despliegue de la primera versión (5 réplicas):

    kubectl create -f service.yaml
    kubectl create -f deploy1.yaml

En un terminal, vemos los pods:

    watch kubectl get pod

En otro terminal, accedemos a la aplicación:

    service=$(minikube service my-app --url)
    while sleep 0.1; do curl "$service"; done

Desplegamos una réplica de la versión 2, para ver si funciona bien:

    kubectl create -f deploy2.yaml

Una vez que comprobamos que funciona bien, podemos escalar y eliminar la versión 1:

    kubectl scale --replicas=5 deploy my-app-v2
    kubectl delete deploy my-app-v1

## Ejemplo 5: ingress

    kubectl create -f nginx-ingress.yaml 
    kubectl get ingress

## Ejemplo 6: LetsChat

    kubectl create -f letschat-deployment.yaml
    kubectl create -f mongo-deployment.yaml
    
    kubectl create -f letschat-srv.yaml
    kubectl create -f mongo-srv.yaml
    
    kubectl create -f ingress.yaml



