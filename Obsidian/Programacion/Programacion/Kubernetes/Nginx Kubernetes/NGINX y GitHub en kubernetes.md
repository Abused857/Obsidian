
# Instalación de Ingress NGINX en Kubernetes

## 1. Instalar o actualizar Ingress NGINX
Para instalar o actualizar el controlador Ingress NGINX, ejecuta el siguiente comando. Asegúrate de tener un archivo de configuración `values.yaml` si necesitas personalizar la instalación:

```
helm upgrade --install -f values.yaml ingress-nginx ingress-nginx \
    --repo https://kubernetes.github.io/ingress-nginx \
    --namespace ingress-nginx \
    --create-namespace
```


### Parámetros del comando:

- **`-f values.yaml`**: Especifica un archivo de valores para personalizar la configuración del chart.
- **`ingress-nginx`**: Nombre de la liberación de Helm.
- **`--repo`**: URL del repositorio donde se encuentra el chart de Ingress NGINX.
- **`--namespace ingress-nginx`**: Define el namespace donde se desplegará el controlador.
- **`--create-namespace`**: Crea el namespace si no existe.

## 2. Verificar el estado del servicio

Después de la instalación, puedes verificar el estado del servicio Ingress NGINX:

`kubectl get services -n ingress-nginx`

Esto te mostrará la dirección IP y otros detalles del servicio.

## 3. Ejemplo de salida

```
NAME     ip-ingress-nginx-prestalo-stg

TYPE     LoadBalancer

CLUSTER-IP     10.96.0.100 

EXTERNAL-IP    XX.XX.XX.XX

PORT(S)    80:XXXX/TCP,443:YYYY/TCP 

AGE    10m


```

Un ejemplo de la salida del comando `kubectl get services` podría ser:


### Detalles de la salida:



- **`ip-ingress-nginx-prestalo-stg`**: Nombre del servicio Ingress NGINX.
- **`EXTERNAL-IP`**: La dirección IP externa asignada al servicio, que se puede utilizar para acceder a tus aplicaciones.

## Resumen

Este proceso configura el controlador Ingress NGINX en tu clúster de Kubernetes, permitiendo gestionar el acceso a tus aplicaciones a través de reglas de enrutamiento.

[[Kubernetes]]