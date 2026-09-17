# Simulacro de Examen CKA 2

Este es el segundo simulacro completo para el examen Certified Kubernetes Administrator (CKA). Consta de 17 preguntas diseñadas para completarse en un tiempo máximo de **2 horas**. 

A diferencia del primer simulacro, estas preguntas cubren escenarios distintos que frecuentemente aparecen en el examen real. 

**Formato:** Primero encontrarás todas las preguntas y luego, al final del documento, sus soluciones detalladas paso a paso. Recuerda configurar el contexto adecuado si estuvieras en un entorno real antes de cada pregunta.

---

## PREGUNTAS (Tiempo estimado: 120 minutos)

### Cluster Architecture, Installation & Configuration (25%)

**Pregunta 1 (7pts) - RBAC**
*Contexto:* `kubectl config use-context cluster1`
Crea un `ClusterRole` llamado `node-pv-reader` que permita listar (`list`) nodos (`nodes`) y volúmenes persistentes (`persistentvolumes`).
Crea un `ClusterRoleBinding` llamado `node-pv-binding` que enlace este rol al `ServiceAccount` llamado `monitoring-sa` ubicado en el namespace `monitoring`. (Si el namespace o el ServiceAccount no existen, créalos).

**Pregunta 2 (6pts) - Certificados de Usuario**
*Contexto:* `kubectl config use-context cluster1`
Tienes un usuario `developer` para el que se ha generado una solicitud de firma de certificado (`csr`).
1. Crea un `CertificateSigningRequest` llamado `developer-csr` usando un archivo base64 que supuestamente está en `/opt/developer.csr` (asume que existe para este ejercicio, o crea un CSR ficticio con el API de k8s).
2. Aprueba el CSR en Kubernetes.
3. Extrae el certificado aprobado y asume que configuras el archivo kubeconfig del usuario. Escribe los comandos para agregar las credenciales y el contexto en kubeconfig.

**Pregunta 3 (6pts) - Mantenimiento de Nodos**
*Contexto:* `kubectl config use-context cluster1`
El nodo `worker-2` requiere mantenimiento. Drena el nodo `worker-2` para que todos los pods sean desalojados de forma segura, asegurándote de que los pods gestionados por `DaemonSet` sean ignorados durante el proceso (no fallen).
Una vez que el nodo esté listo, quítale la restricción para que vuelva a aceptar carga de trabajo.

**Pregunta 4 (6pts) - Salud del Control Plane**
*Contexto:* `kubectl config use-context cluster1`
Verifica el estado de salud de los componentes principales del control plane en el nodo máster (API server, scheduler, controller-manager, etcd). Extrae el comando que usarías para listar los pods estáticos del control plane del namespace `kube-system` y un comando adicional para ver que el servicio en el host (como `kubelet`) está corriendo.

### Workloads & Scheduling (15%)

**Pregunta 5 (5pts) - DaemonSets**
*Contexto:* `kubectl config use-context cluster2`
Crea un `DaemonSet` llamado `log-collector` en el namespace `logging` utilizando la imagen `fluentd:v1.14-1`. (Crea el namespace si no existe). Asegúrate de que los pods del DaemonSet corran en todos los nodos (incluso en el control-plane si es necesario tolerar el taint, aunque para este ejercicio basta con la definición base del DaemonSet y tolerar el taint del máster si se requiere explícitamente).

**Pregunta 6 (5pts) - CronJobs**
*Contexto:* `kubectl config use-context cluster2`
Crea un `CronJob` llamado `hello-cron` en el namespace `default`.
- Debe ejecutarse cada minuto.
- La imagen del pod debe ser `busybox:1.28`.
- El comando a ejecutar en el contenedor debe ser: `sh -c 'date; echo Hello from CKA'`
- Configura el CronJob para que conserve exactamente los últimos 3 trabajos exitosos (successfulJobsHistoryLimit).

**Pregunta 7 (5pts) - Taints & Tolerations**
*Contexto:* `kubectl config use-context cluster2`
1. Aplica un Taint al nodo `node01` con la clave `spray`, el valor `mortein` y el efecto `NoSchedule`.
2. Crea un Pod llamado `frontend-pod` con la imagen `nginx:alpine` y label `tier=frontend`.
3. Añade la toleration correspondiente al Pod para que pueda ser programado en `node01`.

### Services & Networking (20%)

**Pregunta 8 (7pts) - Services & DNS**
*Contexto:* `kubectl config use-context cluster3`
Existe un deployment llamado `backend-app` escuchando en el puerto 8080 en el namespace `web`.
1. Crea un Service de tipo `ClusterIP` llamado `backend-svc` que exponga este deployment por el puerto 80.
2. Crea un pod temporal de prueba usando la imagen `curlimages/curl` y ejecuta un comando `curl` contra el servicio `backend-svc` para comprobar la conectividad.

**Pregunta 9 (7pts) - Network Policies**
*Contexto:* `kubectl config use-context cluster3`
1. Crea una `NetworkPolicy` llamada `default-deny` en el namespace `restricted` que deniegue por defecto todo el tráfico de entrada (Ingress) a todos los pods del namespace.
2. Crea otra `NetworkPolicy` llamada `allow-monitoring` en el mismo namespace que permita el tráfico Ingress hacia los pods con el label `app=db` **solamente** desde los pods en el namespace `monitoring` (asume que el namespace `monitoring` tiene el label `kubernetes.io/metadata.name=monitoring`).

**Pregunta 10 (6pts) - CoreDNS Troubleshooting**
*Contexto:* `kubectl config use-context cluster3`
Diagnostica la resolución DNS interna en el clúster:
1. Escribe el comando para verificar que los pods de CoreDNS están corriendo.
2. Despliega un pod llamado `dns-test` con la imagen `busybox:1.28` e instruye a dicho pod que ejecute `nslookup kubernetes.default` y termine, mostrando el resultado.

### Storage (10%)

**Pregunta 11 (5pts) - PersistentVolumeClaims**
*Contexto:* `kubectl config use-context cluster4`
Crea un `PersistentVolumeClaim` llamado `nginx-pvc` que solicite `500Mi` de almacenamiento, utilice la StorageClass `local-storage` y tenga el access mode `ReadWriteOnce`.
A continuación, crea un Pod llamado `nginx-storage` con la imagen `nginx` que monte este PVC en la ruta `/usr/share/nginx/html`.

**Pregunta 12 (5pts) - Ciclo de vida del volumen**
*Contexto:* `kubectl config use-context cluster4`
1. Crea un `PersistentVolume` llamado `data-pv` de `1Gi` usando `hostPath` (ruta `/mnt/data`), mode `ReadWriteOnce`.
2. Crea un `PersistentVolumeClaim` llamado `data-pvc` de `1Gi` que se enlace al PV anterior.
3. Crea un pod `data-writer` usando la imagen `busybox` que monte el PVC en `/data` y ejecute `echo "persisted" > /data/test.txt; sleep 3600`.
Explica cómo verificarías que los datos persisten si eliminas el pod y creas uno nuevo con el mismo PVC.

### Troubleshooting (30%)

**Pregunta 13 (7pts) - Fix Deployment Image**
*Contexto:* `kubectl config use-context cluster5`
Un deployment llamado `api-server` (en el namespace `default`) fue creado, pero tiene 0/3 pods en estado Ready (CrashLoopBackOff o ImagePullBackOff).
Al inspeccionar, el nombre de la imagen está mal escrito (ej. `ngnix` en vez de `nginx`).
Escribe los comandos para solucionar el problema en vivo y comprobar que el deployment escala correctamente a los 3 pods.

**Pregunta 14 (6pts) - Kubelet Failure**
*Contexto:* `kubectl config use-context cluster5`
El nodo `worker-1` aparece como `NotReady`. Te conectas por SSH al nodo y sospechas que el proceso `kubelet` no está corriendo.
Proporciona los comandos que ejecutarías directamente en el nodo (SSH) para comprobar el estado, habilitar el servicio si estaba deshabilitado y arrancar el kubelet para que el nodo regrese a estado `Ready`.

**Pregunta 15 (6pts) - Static Pod Fix**
*Contexto:* `kubectl config use-context cluster5`
Al entrar por SSH al nodo control-plane, ves que el componente `kube-apiserver` está fallando constantemente (reinicios). Observas un error de YAML en el manifiesto estático en `/etc/kubernetes/manifests/kube-apiserver.yaml`.
Proporciona el comando para editar dicho archivo y el proceso esperado (el kubelet detectará el cambio y reiniciará el pod estático automáticamente).

**Pregunta 16 (5pts) - Búsqueda de Pods (JSONPATH / Custom Columns)**
*Contexto:* `kubectl config use-context cluster5`
Escribe el comando para listar todos los pods del clúster en todos los namespaces, pero que el output esté **ordenado** por su timestamp de creación (`creationTimestamp`).
Guarda la salida en el archivo `/opt/pods-sorted.txt`.

**Pregunta 17 (6pts) - Fix PVC Pending**
*Contexto:* `kubectl config use-context cluster5`
Existe un pod llamado `data-pod` en estado `Pending` porque no puede montar su PVC llamado `missing-pvc`. El PVC también está en estado `Pending`.
Al revisar, el PVC fue solicitado con una `storageClassName` llamada `fast-storage`, la cual no existe. Existe en cambio una clase llamada `standard`.
Escribe los comandos necesarios para eliminar el PVC inválido, crear uno nuevo con la StorageClass correcta y asegurarte de que el pod arranque. (Nota: los PVCs no pueden cambiar su StorageClass en vivo, hay que recrearlos).

---

## SOLUCIONES

### Cluster Architecture, Installation & Configuration

**Solución 1: RBAC**
```bash
# Crear el namespace si no existe
kubectl create namespace monitoring

# Crear el ServiceAccount
kubectl create serviceaccount monitoring-sa -n monitoring

# Crear el ClusterRole
kubectl create clusterrole node-pv-reader --verb=list --resource=nodes,persistentvolumes

# Crear el ClusterRoleBinding
kubectl create clusterrolebinding node-pv-binding \
  --clusterrole=node-pv-reader \
  --serviceaccount=monitoring:monitoring-sa
```

**Solución 2: Certificados de Usuario**
```bash
# 1. Crear el objeto CertificateSigningRequest en Kubernetes
# Asumiendo que el archivo /opt/developer.csr existe:
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: developer-csr
spec:
  request: $(cat /opt/developer.csr | base64 | tr -d "\n")
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF

# 2. Aprobar el CSR
kubectl certificate approve developer-csr

# 3. Extraer el certificado (opcional para visualización) y configurar kubeconfig
# Extraer el cert a archivo
kubectl get csr developer-csr -o jsonpath='{.status.certificate}' | base64 --decode > developer.crt

# Agregar las credenciales al kubeconfig
kubectl config set-credentials developer --client-certificate=developer.crt --client-key=developer.key

# Agregar el contexto
kubectl config set-context developer-context --cluster=kubernetes --user=developer

# Verificar
kubectl config use-context developer-context
```

**Solución 3: Mantenimiento de Nodos**
```bash
# Drenar el nodo ignorando los daemonsets (es mandatorio para nodos que tienen DS)
kubectl drain worker-2 --ignore-daemonsets --force

# Realizar tareas de mantenimiento...

# Una vez finalizado, restaurar el nodo quitándole el cordón (uncordon)
kubectl uncordon worker-2
```

**Solución 4: Salud del Control Plane**
```bash
# Verificar los pods del control plane (estáticos)
kubectl get pods -n kube-system -l tier=control-plane

# Si necesitas ver en detalle el estado de los componentes (dependiendo de la versión de K8s):
kubectl get componentstatuses # (Obsoleto pero útil en versiones viejas)

# Chequeo desde el nodo directamente (vía SSH) para verificar kubelet
systemctl status kubelet
```

### Workloads & Scheduling

**Solución 5: DaemonSets**
```bash
kubectl create namespace logging
```
Crear un archivo `log-collector.yaml`:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  namespace: logging
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1.14-1
      tolerations:
      # Opcional, para que corra en los másters también:
      - effect: NoSchedule
        operator: Exists
```
```bash
kubectl apply -f log-collector.yaml
```

**Solución 6: CronJobs**
```bash
kubectl create cronjob hello-cron \
  --image=busybox:1.28 \
  --schedule="* * * * *" \
  -- dry-run=client -o yaml > cron.yaml
```
Edita `cron.yaml` para añadir `successfulJobsHistoryLimit: 3`:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
  namespace: default
spec:
  schedule: "* * * * *"
  successfulJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello-cron
            image: busybox:1.28
            command:
            - sh
            - -c
            - "date; echo Hello from CKA"
          restartPolicy: OnFailure
```
```bash
kubectl apply -f cron.yaml
```

**Solución 7: Taints & Tolerations**
```bash
# 1. Aplicar Taint
kubectl taint nodes node01 spray=mortein:NoSchedule

# 2 y 3. Crear el Pod con label y toleration (crear pod.yaml)
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
  labels:
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:alpine
  tolerations:
  - key: "spray"
    operator: "Equal"
    value: "mortein"
    effect: "NoSchedule"
```
```bash
kubectl apply -f pod.yaml
```

### Services & Networking

**Solución 8: Services & DNS**
```bash
# Exponer el deployment
kubectl expose deployment backend-app --name=backend-svc --port=80 --target-port=8080 --type=ClusterIP -n web

# Crear pod temporal para probar
kubectl run temp-curl --image=curlimages/curl --restart=Never -it --rm -- /bin/sh
# Dentro del pod temporal ejecutar:
curl http://backend-svc.web.svc.cluster.local
```

**Solución 9: Network Policies**
Crear namespace: `kubectl create namespace restricted`

Crear `default-deny.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: restricted
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```
Crear `allow-monitoring.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-monitoring
  namespace: restricted
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: monitoring
```
```bash
kubectl apply -f default-deny.yaml
kubectl apply -f allow-monitoring.yaml
```

**Solución 10: CoreDNS Troubleshooting**
```bash
# 1. Verificar estado de CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns

# 2. Test DNS
kubectl run dns-test --image=busybox:1.28 --restart=Never --rm -it -- nslookup kubernetes.default
```

### Storage

**Solución 11: PersistentVolumeClaims**
Crear `pvc-pod.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: local-storage
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-storage
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: mypd
  volumes:
  - name: mypd
    persistentVolumeClaim:
      claimName: nginx-pvc
```
```bash
kubectl apply -f pvc-pod.yaml
```

**Solución 12: Ciclo de vida del volumen**
Crear `pv-pvc-pod.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv
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
  name: data-pvc
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
  name: data-writer
spec:
  containers:
  - name: writer
    image: busybox
    command: ["/bin/sh", "-c", "echo 'persisted' > /data/test.txt; sleep 3600"]
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: data-pvc
```
```bash
kubectl apply -f pv-pvc-pod.yaml

# Para comprobar la persistencia:
# kubectl delete pod data-writer
# Crear un nuevo pod "data-reader" que monte el mismo PVC "data-pvc" y ejecutar:
# kubectl exec data-reader -- cat /data/test.txt
```

### Troubleshooting

**Solución 13: Fix Deployment Image**
```bash
# Cambiar la imagen directamente de forma imperativa:
kubectl set image deployment/api-server api-server=nginx:latest

# Alternativamente, puedes editar el deployment:
# kubectl edit deployment api-server

# Verificar que los pods se levantan correctamente
kubectl get pods -w
```

**Solución 14: Kubelet Failure**
```bash
# Entrar al nodo (en el examen se suele usar ssh)
ssh worker-1

# Comprobar el estado del servicio
systemctl status kubelet

# Habilitar el inicio automático (si estuviera desactivado) y arrancarlo
sudo systemctl enable kubelet
sudo systemctl start kubelet

# Salir de la sesión ssh
exit

# Verificar el nodo
kubectl get nodes
```

**Solución 15: Static Pod Fix**
```bash
# Acceder al nodo control-plane
ssh control-plane

# Ir a la ruta de manifiestos
cd /etc/kubernetes/manifests/

# Editar el archivo problemático y corregir el error de indentación o tipográfico
sudo vi kube-apiserver.yaml

# Guardar los cambios. El Kubelet local en el control-plane detecta automáticamente 
# las modificaciones de los archivos en esta carpeta y reinicia el pod estático.

exit
```

**Solución 16: Búsqueda de Pods (JSONPATH / Custom Columns)**
```bash
# Ordenar por creationTimestamp
kubectl get pods -A --sort-by=.metadata.creationTimestamp > /opt/pods-sorted.txt
```

**Solución 17: Fix PVC Pending**
```bash
# 1. Eliminar el PVC erróneo (probablemente tengas que borrar el pod primero o hacer un force, 
# pero un PVC libre se borra sin problema)
kubectl delete pod data-pod
kubectl delete pvc missing-pvc

# 2. Crear un yaml corregido o usar el mandato imperativo si lo permite. 
# Creando un archivo pvc-fix.yaml:
```
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: missing-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
```
```bash
kubectl apply -f pvc-fix.yaml

# 3. Volver a desplegar el pod que usaba 'missing-pvc' (suponiendo que tienes su manifiesto).
# kubectl apply -f data-pod.yaml
```
