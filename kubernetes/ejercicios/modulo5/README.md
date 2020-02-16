# Configuración de aplicaciones

## Ejemplo 1: Variables de entorno

Podemos definir un Deployment que defina un contenedor configurado por medio de variables de entorno.

Creamos el despliegue:

    kubectl create -f mariadb-deployment.yaml

O directamente ejecutando:

    kubectl run mariadb --image=mariadb --env MYSQL_ROOT_PASSWORD=my-password

Veamos el pod creado:

    kubectl get pods -l app=mariadb

Y probamos si podemos acceder, introduciendo la contraseña configurada:

    kubectl exec -it mariadb-deployment-fc75f956-f5zlt -- mysql -u root -p

## Ejemplo 2: ConfigMap

ConfigMap te permite definir un diccionario (clave,valor) para guardar información que puedes utilizar para configurar una aplicación.

    kubectl create cm mariadb --from-literal=root_password=my-password \
                              --from-literal=mysql_usuario=usuario     \
                              --from-literal=mysql_password=password-user \
                              --from-literal=basededatos=test

    kubectl get cm
    kubectl describe cm mariadb

Creamos un deployment indicando los valores guardados en el ConfigMap:

    kubectl create -f mariadb-deployment-configmap.yaml
    kubectl exec -it mariadb-deploy-cm-57f7b9c7d7-ll6pv -- mysql -u usuario -p

## Ejemplo 3: Secrets

Los Secrets nos permiten guardar información sensible que será codificada. Por ejemplo,nos permite guarda contraseñas, claves ssh, …
Al crear un Secret los valores se pueden indicar desde un directorio, un fichero o un literal.

    kubectl create secret generic mariadb --from-literal=password=root
    kubectl get secret
    kubectl describe secret mariadb

Creamos el despliegue y probamos el acceso:

    kubectl create -f mariadb-deployment-secret.yaml
    kubectl exec -it mariadb-deploy-secret-f946dddfd-kkmlb -- mysql -u root -p





