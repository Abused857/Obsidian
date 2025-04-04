

# Autenticación en Google Cloud y Kubernetes

## 1. Autenticación en Google Cloud

- **Iniciar sesión en Google Cloud**:
```
  gcloud auth login
```

Esto abrirá una ventana en tu navegador para que inicies sesión con tu cuenta de Google.

- **Seleccionar el proyecto**: Asegúrate de que estás trabajando en el proyecto correcto:
    
    `gcloud config set project prestalo-infra-stg`
    

## 2. Autenticación en Kubernetes

- **Autenticarse en el clúster de Kubernetes**: Si estás utilizando Google Kubernetes Engine (GKE), asegúrate de que tienes acceso al clúster:
    `gcloud container clusters get-credentials [NOMBRE_DEL_CLÚSTER] --zone [ZONA] --project prestalo-infra-stg`
    
    Reemplaza `[NOMBRE_DEL_CLÚSTER]` y `[ZONA]` con el nombre y la zona de tu clúster.

## Resumen

Antes de ejecutar comandos para crear namespaces y cuentas de servicio, asegúrate de estar autenticado en Google Cloud y en el clúster de Kubernetes.

#  Configuración de Namespace y Servicio


### 1. Crear un espacio de nombres en Kubernetes

`kubectl create namespace external-secrets`

Esto establece un nuevo namespace llamado `external-secrets` para gestionar recursos relacionados con el acceso a secretos externos.

### 2. Crear una cuenta de servicio en Google Cloud

`gcloud iam service-accounts create external-secrets`

Esto crea una nueva cuenta de servicio llamada `external-secrets` para la autenticación y autorización en aplicaciones.

### 3. Asignar permisos a la cuenta de servicio

`gcloud projects add-iam-policy-binding prestalo-infra-stg --member "serviceAccount:external-secrets@xxxxxx.iam.gserviceaccount.com" --role "roles/secretmanager.secretAccessor"`

En el lugar donde aparece `xxxxxx`, debes reemplazarlo con el ID de tu proyecto de Google Cloud. Este ID es único para cada proyecto y tiene el formato de una cadena de texto.

Por ejemplo, si tu proyecto se llama `prestalo-infra-stg`, el correo de la cuenta de servicio debería verse así:

`external-secrets@prestalo-infra-stg.iam.gserviceaccount.com`

Esto le otorga a la cuenta de servicio el rol `secretAccessor`, permitiéndole acceder a los secretos en Google Secret Manager dentro del proyecto `prestalo-infra-stg`.

## Resumen

Estás configurando el acceso a secretos en Google Cloud para que puedan ser utilizados desde tu aplicación en Kubernetes. 


# Configuración de External Secrets en Kubernetes

## 1. Agregar el repositorio de Helm
Para instalar `external-secrets`, primero agrega el repositorio de Helm:
```
helm repo add external-secrets https://charts.external-secrets.io
```

## . Actualizar el repositorio de Helm

Actualiza el repositorio para asegurarte de tener la última versión:


`helm repo update`

## 3. Instalar o actualizar External Secrets

Ejecuta el siguiente comando para instalar o actualizar `external-secrets`, asegurándote de reemplazar `xxxxxxxx` con el ID de tu cuenta de servicio:

```
helm upgrade -install external-secrets external-secrets/external-secrets \
    --set 'serviceAccount.annotations.iam\.gke\.io\/gcp-service-account'="external-secrets@xxxxxxxx.iam.gserviceaccount.com" \
    --namespace external-secrets \
    --create-namespace \
    --debug \
    --wait
```

- **`--set 'serviceAccount.annotations.iam\.gke\.io\/gcp-service-account'`**: Especifica la cuenta de servicio de Google Cloud que se usará para acceder a los secretos.

## 4. Configurar ClusterSecretStore

Crea un archivo YAML para definir un `ClusterSecretStore`. Asegúrate de reemplazar `xxxxxx` con tu ID de proyecto de Google Cloud:

```
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: gcp-store
spec:
  provider:
    gcpsm:
        projectID: xxxxxxx

```
## 5. Configurar ExternalSecret

Crea un archivo YAML para definir un `ExternalSecret`. Este ejemplo obtiene un secreto de Google Secret Manager y crea un secreto en Kubernetes. Asegúrate de que el namespace (`app`) y el secreto se ajusten a tus necesidades:

```
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: test
  namespace: app
spec:
  refreshInterval: 10s            # Intervalo para actualizar los secretos desde GCPSM
  secretStoreRef:
    kind: ClusterSecretStore
    name: gcp-store               # Nombre del ClusterSecretStore definido anteriormente
  target:
    name: test                    # Nombre del secreto de Kubernetes a crear
    creationPolicy: Owner
  data:
  - secretKey: test               # Nombre del key del secreto en Kubernetes
    remoteRef:
      key: test                   # Nombre del secreto en Google Secret Manager
      version: latest              # Versión del secreto (puedes especificar un número de versión si lo deseas)

```
## Resumen

Estos pasos configuran `external-secrets` en Kubernetes, permitiendo que tu aplicación acceda a secretos almacenados en Google Secret Manager.
[[Kubernetes]]