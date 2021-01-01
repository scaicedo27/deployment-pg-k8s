# Despliegue de base de datos Postgres en Kubernetes

Se realizó el despliegue en base a la siguiente url: https://severalnines.com/database-blog/using-kubernetes-deploy-postgresql


## Requisitos

 1.  Tener previamente un cluster de Kubernetes puede ser la versión gratuita o de pago. 
 2.  Tener la CLI de IBM Cloud o acceso al shell online de IBM Cloud.
 
# Pasos
 1. Ingresar por medio de la consola de IBM Cloud enfocado a la cuenta donde esta el cluster de Kubernetes.
 2. Crear conexión directa al cluster deseado con el siguiente código: 
 
```
    ibmcloud ks cluster config --cluster $CLUSTER_NAME
```
3. Probar conexión a la consola de kubectl con el siguiente código:

```
kubectl config current-context
``` 
4.  Crear el primer archivo Yaml del mapa de configuraciones al cluster, donde se definen los datos de acceso principales a la base de datos como son el usuario, la contraseña y la base de datos inicial de conexión. El código que se utiliza se encuentra en el archivo postgres-configmap.yaml. En este archivo se deben realizar los sigueintes cambios dependiendo de la información que se desee: 

```
POSTGRES_DB: ** Nombre base de datos ** 
POSTGRES_USER: ** Usuario de conexión**
POSTGRES_PASSWORD: ** Contraseña **
```
5. Aplicar la configuración al cluster de la siguiente manera:

```
kubectl create -f postgres-configmap.yaml
```

6. Crear el archivo yaml para utilizar y desplegar un volumen persistente de memoria dentro del cluster para la utilización de la base de datos y evitar así la perdida de información. Para esto se utiliza el archivo postgres-storage.yaml, donde se puede editar la cantidad de memoria que se le asigna dentro del cluster en este caso solo se asigna un 1Gb, para cambiarlo se modifican la siguientes lineas: 

```
capacity:
storage: ** Cantidad de memoria deseada** Gi
``` 
y 
```
requests:
storage: ** Cantidad de memoria deseada** Gi
```

7.  Aplicar la configuración al cluster de la siguiente manera:

```
kubectl create -f postgres-storage.yaml
``` 

8. Crear el archivo yaml para el despliegue de la base datos dentro del cluster de kubernetes, para esto utilizamos el archivo que se llama postgres-deployment.yaml. Aquí se establce el puerto de conexión, la imagen de base del postgres y la versión del mismo. 

9.  Aplicar la configuración al cluster de la siguiente manera:
```
kubectl create -f postgres-deployment.yaml
``` 

10. Por último se crea el archivo de exposición del servicio de la base de datos de postgres dentro del cluster de Kubernetes, por medio del uso del archivo postgres-service.yaml. Acá se establece las conexión externa a la base de datos, como exposición del puerto y de la ruta. Además de ser necesario se puede crear un balanceador de cargas para el acceso a la misma. 

11. Aplicar la configuración al cluster de la siguiente manera:
```
kubectl create -f postgres-service.yaml
``` 

## Conexión a la base de datos

Para conectarnos a la base ya desplegada desde la consola obtenemos el puerto del servicio donde se creo el despliegue con el siguiente codigo: 
```
kubectl get svc postgres
```
Al correr el comando nos arroja el siguiente resultado y se toma el puerto que acompaña al 5432, para construir la ruta de acceso. En este caso es el puerto 31070.
```
$ kubectl get svc postgres
NAME       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
postgres   NodePort   10.107.71.xx   <none>        5432:31070/TCP   5m
```
Por último obtenemos la ruta pública del cluster con el comando: 
```
ibmcloud cs workers --cluster **Nombre del cluster**
```
La respuesta es la siguiente: 
```
ID           Public IP        Private IP      Flavor   State    Status   Zone    Version   
id_cluster   192.120.170.xx   10.124.204.xx   free     normal   Ready    mil01   1.18.13_1536   
```
La ruta final , que es la que se utiliza para la conexión con otras aplicaciones, se construye tomando la ruta publica del cluster y asignando el puerto que obtuvimos previamente al final quedado una ruta como la siguiente:

```
192.120.170.xx:31070
```
