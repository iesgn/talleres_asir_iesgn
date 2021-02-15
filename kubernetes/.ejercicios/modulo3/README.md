# Módulo 3: Depliegue de aplicaciones en Kubernetes

## Ejemplo 1: Pod

Para crear el pod desde el fichero yaml:

    kubectl create -f pod.yaml

Y podemos ver que el pod se ha creado:

    kubectl get pods

Si queremos saber en qué nodo del cluster se está ejecutando:

    kubectl get pod -o wide

Para obtener información más detallada del pod:

    kubectl describe pod nginx

Para eliminar el pod:

    kubectl delete pod nginx

Podemos modificar las características de cualquier recurso de kubernetes una vez creado, por ejemplo podemos modificar la definición del pod de la siguiente manera:

    kubectl edit pod nginx
    KUBE_EDITOR="nano" kubectl edit pod nginx

Para obtener los logs del pod:

    kubectl logs nginx

Si quiero conectarme al contenedor:

    kubectl exec -it nginx -- /bin/bash

Podemos acceder a la aplicación, redirigiendo un puerto de localhost al puerto de la aplicación:

    kubectl port-forward nginx 8080:80

Y accedemos al servidor web en la url `http://localhost:8080`.

Para obtener las labels de los pods que hemos creado:

    kubectl get pods --show-labels

Los Labels lo hemos definido en la sección metada del fichero yaml, pero también podemos añadirlos a los pods ya creados:

    kubectl label pods nginx service=web

Los Labels me van a permitir seleccionar un recurso determinado, por ejemplo para visualizar los pods que tienen un label con un determinado valor:

    kubectl get pods -l service=web

También podemos visualizar los valores de los labels como una nueva columna:

    kubectl get pods -Lservice

## Ejemplo 2: Pod multicontenedor

    kubectl create -f  pod-multi-container.yaml

    kubectl describe pod mc1

    kubectl exec mc1 -c 1st -- /bin/cat /usr/share/nginx/html/index.html
  
    kubectl exec mc1 -c 2nd -- /bin/cat /html/index.html

    kubectl port-forward mc1 8080:80

## Ejemplo 3: ReplicaSet

Creamos el ReplicaSet:
    
    kubectl create -f nginx-rs.yaml

    kubectl get rs
    kubectl get pods

¿Qué pasaría si borro uno de los pods que se han creado? Inmediatamente se creará uno nuevo para que siempre estén ejecutándose los pods deseados, en este caso 2:

    kubectl delete pod nginx-5b2rn
    kubectl get pods

Para escalar el número de pods:

    kubectl scale rs nginx --replicas=5
    kubectl get pods --watch

Como anteriormente vimos podemos modificar las características de un ReplicaSet con la siguiente instrucción:

    kubectl edit rs nginx

Por último si borramos un ReplicaSet se borraran todos los pods asociados:

    kubectl delete rs nginx

## Ejemplo 4: Deployment

Cuando creamos un Deployment, se crea el ReplicaSet asociado y todos los pods que hayamos indicado.

    kubectl create -f nginx-deployment.yaml
    kubectl get deploy,rs,pod

Como ocurría con los replicaSets los Deployment también se pueden escalar, aumentando o disminuyendo el número de pods asociados:

    kubectl scale deployment nginx --replicas=4

Otras operaciones:

    kubectl port-forward deploy/nginx 8080:80
    kubectl logs deploy/nginx

Si eliminamos el Deployment se eliminarán el ReplicaSet asociado y los pods que se estaban gestionando.

    kubectl delete deployment nginx

Para modificar un Deployment (o cualquier objeto k8s):

* Modificando el parámetro directamente. `kubectl set`
* Modificando el fichero yaml y aplicando el cambio. `kubectl apply -f`
* Modificando la definición del objeto: `kubectl edit deployment ...`

Por ejemplo para hacer una actualización de la aplicación:

* `kubectl set image deployment nginx nginx=nginx:1.16 --all`
* Modifico el fichero deployment.yaml y ejecuto:
    * `kubectl apply -f deployment.yaml`
* `kubectl edit deployment nginx`

Actualización y rollout de la aplicación:

    kubectl set image deployment nginx nginx=nginx:1.16 --all

Comprobamos que se ha creado un nuevo ReplicaSet, y unos nuevos pods con la nueva versión de la imagen.

    kubectl get rs
    kubectl get pods

La opción `--all` fuerza a actualizar todos los pods aunque no estén inicializados.

Si queremos volver a la versión anterior de nuestro despliegue, tenemos que ejecutar:

    kubectl rollout undo deployment nginx

Y comprobamos como se activa el antiguo ReplicaSet y se crean nuevos pods con la versión anterior de nuestra aplicación:

    kubectl get rs

## Ejemplo 5: mediaWiki

Vamos a desplegar la aplicación mediawiki:

    kubectl apply -f mediawiki-deploy.yaml --record
    kubectl get all

Comprobamos el historial de actualizaciones y desplegamos una nueva versión:

    kubectl rollout history deployment/mediawiki
    kubectl set image deployment/mediawiki mediawiki=mediawiki:1.27 --all --record
    kubectl get all

Ahora vamos a deplegar una versión que da un error (versión no existe). ¿Poedemos volver al despliegue anterior?

    kubectl rollout history deployment/mediawiki
    kubectl set image deployment/mediawiki mediawiki=mediawiki:2 --all --record
    kubectl rollout undo deployment/mediawiki
    kubectl get all

## Ejemplo 6: guestbook (parte 1)

    kubectl create -f frontend-deployment.yaml
    kubectl create -f redis-master-deployment.yaml
    kubectl create -f redis-slave-deployment.yaml

    kubectl port-forward deployment/guestbook 3000:3000


