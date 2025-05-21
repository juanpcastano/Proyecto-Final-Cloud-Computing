# Patrones De tolerancia a fallos en Kubernetes

## Requisitos previos

- Tener instalado **Minikube** y **kubectl**
- Tener instalado **Helm**

---

## Iniciar Minikube

```bash
minikube start
```

## Aplicar los archivos YAML

```bash
kubectl apply -f postgres.yaml
kubectl apply -f path-management.yaml
kubectl apply -f auth-service.yaml
kubectl apply -f user-management.yaml
kubectl apply -f ai-service.yaml
kubectl apply -f smtp-service.yaml
kubectl apply -f api-gateway.yaml
```


## Verificar estado de los pods y servicios

```bash
kubectl get pods
kubectl get svc
```

### Ejemplo de pods corriendo:

```
NAME                               READY   STATUS    RESTARTS      AGE
ai-service-6b678cd9c6-ch7mc        1/1     Running   0             36m
ai-service-6b678cd9c6-q5psp        1/1     Running   0             37m
api-gateway-59768fd595-bn2h9       1/1     Running   0             37m
api-gateway-59768fd595-kzxfl       1/1     Running   0             36m
auth-service-5fbdc9747-4xqcr       1/1     Running   1 (35m ago)   37m
path-management-c7c5f979b-4grqh    1/1     Running   1 (35m ago)   37m
postgres-78c79d9449-lkj58          1/1     Running   0             37m
smtp-service-596f7459ff-tx2n9      1/1     Running   0             37m
user-management-75f9675868-tkl5b   1/1     Running   0             37m
```

---

## Obtener URL del API Gateway

```bash
minikube service api-gateway --url
```

Usa esta URL para realizar peticiones desde Postman.

> **Nota:** Si estás usando una máquina virtual, ejecuta lo siguiente para exponer el API Gateway hacia el host:
```bash
kubectl port-forward --address 0.0.0.0 service/api-gateway 8090:3000
```
Accede desde: `http://<ip-maquina-virtual>:8090`

---

## Instalar Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

---

## Instalar Litmus Chaos

### Verificar existencia de pods en el namespace `litmus`

```bash
kubectl get pods -n litmus
kubectl get svc -n litmus
```

### Instalar Litmus usando Helm

```bash
helm repo add litmuschaos https://litmuschaos.github.io/litmus-helm/
helm repo list

kubectl create ns litmus

helm install chaos litmuschaos/litmus --namespace=litmus \
--set portal.frontend.service.type=NodePort \
--set portal.server.graphqlServer.genericEnv.CHAOS_CENTER_UI_ENDPOINT=http://chaos-litmus-frontend-service.litmus.svc.cluster.local:9091/
```

### Acceder al portal de Litmus

```bash
minikube service chaos-litmus-frontend-service -n litmus
```

---

## Instalar Prometheus y Grafana

### Crear namespace de monitoreo

```bash
kubectl create namespace monitoring
```

### Instalar Prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/prometheus --namespace monitoring
```

Verificar pods:

```bash
kubectl get pods -n monitoring
```

Exponer Prometheus:

```bash
kubectl expose service prometheus-server --namespace monitoring --type=NodePort --target-port=9090 --name=prometheus-server-ext
```

---

### Instalar Grafana

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install grafana grafana/grafana --namespace monitoring
```

Obtener contraseña de administrador:

```bash
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Exponer servicio de Grafana:

```bash
kubectl expose service grafana --namespace monitoring --type=NodePort --target-port=3000 --name=grafana-ext
```

---

## Acceder a Prometheus y Grafana

Ejecuta:

```bash
minikube ip
```

Y usa la IP obtenida junto con los puertos expuestos:

- Prometheus: `http://<minikube-ip>:31257`
- Grafana: `http://<minikube-ip>:30485`

---

¡Listo! Ya tienes desplegado tu entorno con servicios, monitoreo y caos testing en Minikube y monitoreado.
