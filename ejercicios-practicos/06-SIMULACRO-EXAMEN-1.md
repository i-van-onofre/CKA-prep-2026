# Simulacro de Examen CKA 1 - Prueba Completa

Este es un simulacro completo del examen Certified Kubernetes Administrator (CKA). Consta de 17 preguntas diseñadas para ser resueltas en un máximo de **2 horas**. 

## Instrucciones Generales
- **Duración:** 120 minutos.
- **Puntaje de aprobación:** 66% (66/100 puntos).
- **Recursos permitidos:** Durante el examen real, puedes consultar la documentación oficial (kubernetes.io/docs, github.com/kubernetes, etc.).
- **Contextos:** Antes de cada pregunta, asegúrate de configurar el contexto adecuado tal como se indica.

---

## SECCIÓN DE PREGUNTAS

### Cluster Architecture, Installation & Configuration (25%)

**Pregunta 1 (7 puntos)**
Contexto: `kubectl config use-context k8s-cluster1`
Crea un `Role` llamado `dev-role` y un `RoleBinding` llamado `dev-binding` en el namespace `development`.
Estos recursos deben permitir al usuario `john` crear y listar `pods` y `services` en dicho namespace.

**Pregunta 2 (7 puntos)**
Contexto: `kubectl config use-context k8s-cluster1`
1. Realiza un backup (snapshot) de etcd y guárdalo en `/opt/etcd-backup.db`.
2. Supón que existe un snapshot previo en `/opt/etcd-snapshot-previous.db`. Demuestra el comando (o pasos) que utilizarías para restaurar el clúster desde ese snapshot previo. (En este simulacro, escribe los comandos en un archivo de texto `/opt/etcd-restore-steps.txt`).

**Pregunta 3 (5 puntos)**
Contexto: `kubectl config use-context k8s-cluster1`
Actualiza el nodo del control plane (asumiendo que se llama `controlplane`) a la siguiente versión menor estable de Kubernetes (por ejemplo, si estás en 1.29.0, actualiza a 1.29.1 o 1.30.0 dependiendo de lo disponible) usando `kubeadm`. 
Escribe los comandos exactos que ejecutarías en un archivo `/opt/upgrade-controlplane.txt`.

**Pregunta 4 (6 puntos)**
Contexto: `kubectl config use-context k8s-cluster1`
Crea una `NetworkPolicy` llamada `allow-trusted` en el namespace `secure`.
La política debe permitir tráfico de entrada (Ingress) en todos los puertos, pero **solo** si el tráfico proviene de Pods ubicados en el namespace `trusted`. Deniega el resto del tráfico Ingress a los pods del namespace `secure`.

### Workloads & Scheduling (15%)

**Pregunta 5 (5 puntos)**
Contexto: `kubectl config use-context k8s-cluster2`
Crea un Deployment llamado `web-deploy` en el namespace `default` con la imagen `nginx:1.23` y 3 réplicas.
Configura la estrategia de actualización del Deployment (Rolling Update) para que el `maxSurge` sea 1 y el `maxUnavailable` sea 0.

**Pregunta 6 (5 puntos)**
Contexto: `kubectl config use-context k8s-cluster2`
Crea un Pod llamado `init-pod` con la imagen principal `busybox` (comando `sleep 3600`).
Configura un `initContainer` usando la imagen `busybox` que ejecute el comando `sh -c 'echo "Iniciado" > /workdir/status.txt'`.
Monta un volumen `emptyDir` en `/workdir` en ambos contenedores para que el contenedor principal pueda leer el archivo `/workdir/status.txt`.

**Pregunta 7 (5 puntos)**
Contexto: `kubectl config use-context k8s-cluster2`
Programa un Pod llamado `ssd-pod` con la imagen `redis` en un nodo específico que tenga la etiqueta `disktype=ssd`.
Si no hay nodos con esa etiqueta, no importa si el Pod queda en estado Pending.

### Services & Networking (20%)

**Pregunta 8 (7 puntos)**
Contexto: `kubectl config use-context k8s-cluster3`
Crea un recurso Ingress llamado `app-ingress` en el namespace `default`.
Configura el enrutamiento basado en rutas (path-based routing) de la siguiente manera:
- Tráfico a `/app1` debe ser dirigido al servicio `svc-app1` en el puerto `80`.
- Tráfico a `/app2` debe ser dirigido al servicio `svc-app2` en el puerto `80`.
(Asume que los servicios ya existen).

**Pregunta 9 (7 puntos)**
Contexto: `kubectl config use-context k8s-cluster3`
Crea una `NetworkPolicy` llamada `allow-app-access` en el namespace `default`.
La política debe aplicarse a los Pods que tengan la etiqueta `app=database`.
Debe permitir tráfico Ingress al puerto TCP `3306` **solo** desde Pods que tengan la etiqueta `access=true`.

**Pregunta 10 (6 puntos)**
Contexto: `kubectl config use-context k8s-cluster3`
Se tiene un Deployment llamado `frontend`. Exponlo creando un Service tipo `NodePort` llamado `frontend-svc`.
El servicio debe mapear el puerto 80 del contenedor a un puerto NodePort aleatorio.
Guarda la IP de un nodo del clúster y el NodePort asignado en el archivo `/opt/frontend-access.txt` (formato IP:PORT).

### Storage (10%)

**Pregunta 11 (5 puntos)**
Contexto: `kubectl config use-context k8s-cluster4`
1. Crea un PersistentVolume (PV) llamado `local-pv` de 1Gi, con modo de acceso `ReadWriteOnce`, usando el tipo `hostPath` en `/mnt/data`.
2. Crea un PersistentVolumeClaim (PVC) llamado `local-pvc` que solicite 1Gi y `ReadWriteOnce`.
3. Crea un Pod llamado `storage-pod` con la imagen `nginx` que monte el PVC en la ruta `/usr/share/nginx/html`.

**Pregunta 12 (5 puntos)**
Contexto: `kubectl config use-context k8s-cluster4`
Crea un Pod llamado `shared-data` con dos contenedores:
- Contenedor 1: nombre `writer`, imagen `busybox`, comando `sh -c 'while true; do date >> /shared/date.txt; sleep 5; done'`
- Contenedor 2: nombre `reader`, imagen `busybox`, comando `sh -c 'tail -f /shared/date.txt'`
Ambos deben montar un volumen `emptyDir` en la ruta `/shared`.

### Troubleshooting (30%)

**Pregunta 13 (7 puntos)**
Contexto: `kubectl config use-context k8s-cluster5`
Un Pod llamado `web-app` en el namespace `production` está en estado `CrashLoopBackOff`.
Diagnostica la causa del problema y arréglalo. (En un escenario real examinarías los logs. Para el simulacro, asume que el comando del Pod estaba mal escrito en su YAML: `nginx-start` en lugar de `nginx`. Escribe los pasos para solucionarlo).

**Pregunta 14 (6 puntos)**
Contexto: `kubectl config use-context k8s-cluster5`
Un nodo worker llamado `node01` está en estado `NotReady`.
Escribe los comandos que utilizarías para diagnosticar y recuperar el nodo. Guárdalos en `/opt/node01-troubleshoot.txt`. (Pista: suele ser kubelet detenido).

**Pregunta 15 (6 puntos)**
Contexto: `kubectl config use-context k8s-cluster5`
Un Service llamado `web-svc` no está dirigiendo el tráfico a los pods de backend.
Diagnostica y soluciona el problema. (Asume que el selector del servicio apunta a `app=frontend` pero los pods tienen la etiqueta `app=webapp`. Explica cómo lo arreglarías).

**Pregunta 16 (5 puntos)**
Contexto: `kubectl config use-context k8s-cluster5`
Identifica el Pod que consume más CPU en todo el clúster.
Redirige el nombre del Pod y su namespace al archivo `/opt/high-cpu-pod.txt`.

**Pregunta 17 (6 puntos)**
Contexto: `kubectl config use-context k8s-cluster5`
Los pods en el namespace `dns-test` no pueden resolver nombres DNS internos (como `kubernetes.default`).
Identifica el problema y describe los pasos para solucionarlo. (Asume que los pods de CoreDNS en `kube-system` no están corriendo. Explica cómo diagnosticar esto).

---

## SECCIÓN DE SOLUCIONES

### Solución 1 (RBAC)
```bash
kubectl config use-context k8s-cluster1

# Crear Role
kubectl create role dev-role -n development --verb=create,list --resource=pods,services

# Crear RoleBinding
kubectl create rolebinding dev-binding -n development --role=dev-role --user=john
```

### Solución 2 (Backup y Restore etcd)
**Backup:**
```bash
kubectl config use-context k8s-cluster1

ETCDCTL_API=3 etcdctl snapshot save /opt/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

**Restore (Pasos en /opt/etcd-restore-steps.txt):**
```bash
# 1. Restaurar el snapshot en un nuevo directorio de datos
ETCDCTL_API=3 etcdctl snapshot restore /opt/etcd-snapshot-previous.db \
  --data-dir=/var/lib/etcd-restore

# 2. Editar /etc/kubernetes/manifests/etcd.yaml
# Modificar el hostPath del volumen 'etcd-data' de /var/lib/etcd a /var/lib/etcd-restore
# 3. Kubelet reiniciará automáticamente el pod de etcd.
```

### Solución 3 (Upgrade Control Plane)
```text
# En /opt/upgrade-controlplane.txt
# 1. Drenar el nodo de control plane
kubectl drain controlplane --ignore-daemonsets

# 2. Actualizar kubeadm (ej. en Ubuntu/Debian)
apt-mark unhold kubeadm
apt-get update && apt-get install -y kubeadm=1.29.x-00
apt-mark hold kubeadm

# 3. Aplicar plan de actualización
kubeadm upgrade plan
kubeadm upgrade apply v1.29.x

# 4. Actualizar kubelet y kubectl
apt-mark unhold kubelet kubectl
apt-get update && apt-get install -y kubelet=1.29.x-00 kubectl=1.29.x-00
apt-mark hold kubelet kubectl
systemctl daemon-reload
systemctl restart kubelet

# 5. Hacer el nodo programable nuevamente
kubectl uncordon controlplane
```

### Solución 4 (NetworkPolicy)
```yaml
# policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-trusted
  namespace: secure
spec:
  podSelector: {} # Aplica a todos los pods en 'secure'
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: trusted
```
Aplicar con `kubectl apply -f policy.yaml`.

### Solución 5 (Deployment Strategy)
```bash
kubectl config use-context k8s-cluster2

# Generar YAML base
kubectl create deployment web-deploy --image=nginx:1.23 --replicas=3 -n default --dry-run=client -o yaml > deploy.yaml
```
Editar `deploy.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
  namespace: default
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: web-deploy
  template:
    metadata:
      labels:
        app: web-deploy
    spec:
      containers:
      - name: nginx
        image: nginx:1.23
```
Aplicar: `kubectl apply -f deploy.yaml`

### Solución 6 (Init Container)
```yaml
# init-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'echo "Iniciado" > /workdir/status.txt']
    volumeMounts:
    - name: workdir
      mountPath: /workdir
  containers:
  - name: main-container
    image: busybox
    command: ['sleep', '3600']
    volumeMounts:
    - name: workdir
      mountPath: /workdir
  volumes:
  - name: workdir
    emptyDir: {}
```
`kubectl apply -f init-pod.yaml`

### Solución 7 (Node Selector)
```yaml
# ssd-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ssd-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: redis
    image: redis
```
`kubectl apply -f ssd-pod.yaml`

### Solución 8 (Ingress)
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: default
spec:
  rules:
  - http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: svc-app1
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: svc-app2
            port:
              number: 80
```
`kubectl apply -f ingress.yaml`

### Solución 9 (NetworkPolicy Port/Label)
```yaml
# np-db.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-access
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true"
    ports:
    - protocol: TCP
      port: 3306
```
`kubectl apply -f np-db.yaml`

### Solución 10 (Exponer Deployment a NodePort)
```bash
kubectl config use-context k8s-cluster3

# Exponer el Deployment
kubectl expose deployment frontend --name=frontend-svc --type=NodePort --port=80

# Averiguar puerto y nodo
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
NODE_PORT=$(kubectl get svc frontend-svc -o jsonpath='{.spec.ports[0].nodePort}')

echo "$NODE_IP:$NODE_PORT" > /opt/frontend-access.txt
```

### Solución 11 (PV y PVC)
```yaml
# pv-pvc-pod.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: storage
      mountPath: /usr/share/nginx/html
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: local-pvc
```
`kubectl apply -f pv-pvc-pod.yaml`

### Solución 12 (EmptyDir)
```yaml
# shared-vol.yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-data
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'while true; do date >> /shared/date.txt; sleep 5; done']
    volumeMounts:
    - name: shared-storage
      mountPath: /shared
  - name: reader
    image: busybox
    command: ['sh', '-c', 'tail -f /shared/date.txt']
    volumeMounts:
    - name: shared-storage
      mountPath: /shared
  volumes:
  - name: shared-storage
    emptyDir: {}
```
`kubectl apply -f shared-vol.yaml`

### Solución 13 (Troubleshooting Pod)
```bash
kubectl config use-context k8s-cluster5

# Diagnóstico:
kubectl describe pod web-app -n production
kubectl logs web-app -n production

# Solución (Corregir el comando de la imagen):
# Extraer el YAML
kubectl get pod web-app -n production -o yaml > web-app.yaml
# Editar web-app.yaml para corregir el comando (cambiar nginx-start por nginx o eliminar el comando si la imagen ya lo tiene por defecto)
# Recrear el pod:
kubectl delete pod web-app -n production
kubectl apply -f web-app.yaml
```

### Solución 14 (Troubleshooting Node)
```text
# En /opt/node01-troubleshoot.txt
# Diagnóstico:
kubectl describe node node01

# Acceder al nodo por SSH:
ssh node01

# Comprobar el servicio kubelet:
systemctl status kubelet
journalctl -u kubelet

# Solución (si está detenido):
systemctl restart kubelet
systemctl enable kubelet
```

### Solución 15 (Troubleshooting Service)
```bash
kubectl config use-context k8s-cluster5

# Diagnóstico (verificar endpoints):
kubectl get endpoints web-svc

# Solución: El Service no encuentra los Pods porque el selector no coincide con las etiquetas de los Pods.
# Editar el Service:
kubectl edit svc web-svc

# Cambiar `app: frontend` a `app: webapp` en la sección de `selector`.
# Guardar y salir. Los endpoints deberían actualizarse automáticamente.
```

### Solución 16 (Top CPU Pod)
```bash
kubectl config use-context k8s-cluster5

# Usar kubectl top para encontrar el que más CPU usa
kubectl top pods -A --sort-by='cpu'

# Si el primero es el pod "metrics-pod" en "monitoring":
echo "monitoring metrics-pod" > /opt/high-cpu-pod.txt
```

### Solución 17 (Troubleshooting DNS)
```bash
kubectl config use-context k8s-cluster5

# Diagnóstico DNS:
kubectl exec -it <nombre-de-un-pod-en-dns-test> -n dns-test -- nslookup kubernetes.default

# Revisar CoreDNS:
kubectl get pods -n kube-system -l k8s-app=kube-dns
# Si no están corriendo, ver sus logs o descripción:
kubectl describe pod -n kube-system -l k8s-app=kube-dns

# Solución posible si se eliminaron los pods:
# Los pods son gestionados por un Deployment, simplemente se recuperarán automáticamente si el nodo tiene recursos, o reiniciar el deployment:
kubectl rollout restart deployment coredns -n kube-system
```
