# Guía Básica de Azure Kubernetes Service (AKS)

## 1. Preparación del Entorno
### Requisitos Previos
- Azure CLI instalado
- Cuenta de Azure activa
- Subscription configurada
- Resource Group creado

### Comandos Iniciales
```bash
# Login a Azure
az login

# Seleccionar subscription
az account set --subscription <subscription-id>

# Crear resource group (si no existe)
az group create --name miRG-aks --location eastus
```

## 2. Creación del Cluster AKS
```bash
# Crear cluster básico
az aks create \
    --resource-group miRG-aks \
    --name miClusterAKS \
    --node-count 2 \
    --enable-addons monitoring \
    --generate-ssh-keys
```

## 3. Configuración de Acceso
```bash
# Obtener credenciales
az aks get-credentials \
    --resource-group miRG-aks \
    --name miClusterAKS

# Verificar conexión
kubectl get nodes
```

## 4. Gestión de Secretos
### Crear Secrets
```bash
# Crear secret desde literal
kubectl create secret generic mi-secreto \
    --from-literal=username=admin \
    --from-literal=password=miPassword123

# Crear secret desde archivo
kubectl create secret generic tls-secret \
    --from-file=tls.crt=path/to/cert.crt \
    --from-file=tls.key=path/to/cert.key
```

## 5. Despliegue de Aplicaciones
### Ejemplo de Deployment Básico
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mi-app
  template:
    metadata:
      labels:
        app: mi-app
    spec:
      containers:
      - name: mi-app
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Comandos de Despliegue
```bash
# Aplicar deployment
kubectl apply -f deployment.yaml

# Verificar estado
kubectl get deployments
kubectl get pods
```

## 6. Servicios
### Crear Servicio
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-app-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: mi-app
```

```bash
# Aplicar servicio
kubectl apply -f service.yaml

# Verificar servicio
kubectl get services
```

## 7. Storage
### Crear PersistentVolumeClaim
```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mi-azure-disk
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: managed-premium
  resources:
    requests:
      storage: 5Gi
```

### Usar Storage en Deployment
```yaml
# deployment-con-storage.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-app-con-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mi-app-storage
  template:
    metadata:
      labels:
        app: mi-app-storage
    spec:
      containers:
      - name: mi-app
        image: nginx:latest
        volumeMounts:
        - name: mi-volumen
          mountPath: "/data"
      volumes:
      - name: mi-volumen
        persistentVolumeClaim:
          claimName: mi-azure-disk
```

## 8. Monitoreo Básico
```bash
# Ver logs de pods
kubectl logs <nombre-pod>

# Describir recursos
kubectl describe pod <nombre-pod>
kubectl describe service <nombre-servicio>

# Monitorear recursos
kubectl top nodes
kubectl top pods
```

## 9. Limpieza
```bash
# Eliminar recursos
kubectl delete deployment mi-app
kubectl delete service mi-app-service
kubectl delete pvc mi-azure-disk

# Eliminar cluster
az aks delete \
    --resource-group miRG-aks \
    --name miClusterAKS \
    --yes --no-wait
```
