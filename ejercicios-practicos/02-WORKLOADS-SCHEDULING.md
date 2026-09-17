# CKA - Ejercicios Prácticos: Workloads & Scheduling (15%)

Este documento contiene 14 ejercicios prácticos enfocados en el dominio de Workloads & Scheduling, fundamentales para el examen Certified Kubernetes Administrator (CKA).

---

## Ejercicio 1: Crear Deployment y escalar
**Tiempo estimado:** 3 minutos

**Enunciado:**
Crea un deployment llamado `nginx-deploy` utilizando la imagen `nginx:1.19` con 3 réplicas en el namespace `default`.
Una vez creado, escala el deployment a 5 réplicas.

**Solución:**

1. Crear el deployment de forma imperativa:
```bash
kubectl create deployment nginx-deploy --image=nginx:1.19 --replicas=3
```
*Nota: En versiones recientes de kubectl, `--replicas` puede no estar disponible en `create deployment`. En ese caso:*
```bash
kubectl create deployment nginx-deploy --image=nginx:1.19 --dry-run=client -o yaml > deploy.yaml
```
Edita `deploy.yaml` para añadir `replicas: 3`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-deploy
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.19
        name: nginx
        resources: {}
status: {}
```
```bash
kubectl apply -f deploy.yaml
```

2. Escalar el deployment a 5 réplicas:
```bash
kubectl scale deployment nginx-deploy --replicas=5
```

**Verificación:**
```bash
kubectl get deployments nginx-deploy
kubectl get pods -l app=nginx-deploy
```
*Doc URL:* [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

---

## Ejercicio 2: Rolling Update y Rollback
**Tiempo estimado:** 5 minutos

**Enunciado:**
Utilizando el deployment `nginx-deploy` del ejercicio anterior, actualiza la imagen a `nginx:1.21`.
Registra la actualización en el historial. Verifica el historial de revisiones del deployment.
Finalmente, realiza un rollback a la revisión anterior.

**Solución:**

1. Actualizar la imagen:
```bash
kubectl set image deployment/nginx-deploy nginx=nginx:1.21 --record
```
*(Nota: `--record` está obsoleto en versiones recientes, pero aún funciona o puedes usar `kubectl annotate` para registrar causas si lo prefieres)*

2. Verificar el historial:
```bash
kubectl rollout history deployment nginx-deploy
```

3. Hacer rollback a la revisión anterior:
```bash
kubectl rollout undo deployment nginx-deploy
```

**Verificación:**
```bash
kubectl describe deployment nginx-deploy | grep Image
kubectl rollout status deployment nginx-deploy
```
*Doc URL:* [Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)

---

## Ejercicio 3: Configurar recursos (requests/limits)
**Tiempo estimado:** 4 minutos

**Enunciado:**
Crea un pod llamado `resource-pod` con la imagen `redis`.
Configura el contenedor para que solicite (request) 100m de CPU y 128Mi de RAM, y tenga un límite (limit) de 200m de CPU y 256Mi de RAM.

**Solución:**

1. Generar la plantilla base:
```bash
kubectl run resource-pod --image=redis --dry-run=client -o yaml > resource-pod.yaml
```

2. Modificar el archivo `resource-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: redis
    image: redis
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```

3. Crear el pod:
```bash
kubectl apply -f resource-pod.yaml
```

**Verificación:**
```bash
kubectl describe pod resource-pod | grep -A 4 Requests:
```
*Doc URL:* [Resource Management for Pods](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

---

## Ejercicio 4: ConfigMap como variable de entorno
**Tiempo estimado:** 4 minutos

**Enunciado:**
Crea un ConfigMap llamado `app-config` con las siguientes llaves y valores: `APP_COLOR=blue` y `APP_MODE=prod`.
Crea un pod llamado `env-pod` con la imagen `busybox` que ejecute el comando `env`.
El pod debe cargar todas las variables de entorno definidas en `app-config`.

**Solución:**

1. Crear el ConfigMap imperativamente:
```bash
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
```

2. Crear el YAML del pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["env"]
    envFrom:
    - configMapRef:
        name: app-config
```
Guardar como `env-pod.yaml` y aplicar:
```bash
kubectl apply -f env-pod.yaml
```

**Verificación:**
```bash
kubectl logs env-pod | grep APP
```
*Doc URL:* [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables)

---

## Ejercicio 5: ConfigMap como volumen
**Tiempo estimado:** 5 minutos

**Enunciado:**
Crea un archivo llamado `config.txt` con el texto `key=value`.
Crea un ConfigMap llamado `file-config` desde este archivo.
Despliega un pod llamado `vol-pod` usando la imagen `nginx`. Monta el ConfigMap en la ruta `/etc/app-config` dentro del pod.

**Solución:**

1. Crear el archivo y el ConfigMap:
```bash
echo "key=value" > config.txt
kubectl create configmap file-config --from-file=config.txt
```

2. Crear el YAML del pod `vol-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vol-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-vol
      mountPath: /etc/app-config
  volumes:
  - name: config-vol
    configMap:
      name: file-config
```
3. Aplicar:
```bash
kubectl apply -f vol-pod.yaml
```

**Verificación:**
```bash
kubectl exec vol-pod -- cat /etc/app-config/config.txt
```
*Doc URL:* [Add ConfigMap data to a Volume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume)

---

## Ejercicio 6: Secret
**Tiempo estimado:** 6 minutos

**Enunciado:**
Crea un Secret de tipo genérico (Opaque) llamado `db-secret` con las llaves `db-user=admin` y `db-pass=password123`.
Crea un pod llamado `secret-pod` (imagen `nginx`) que:
- Inyecte `db-user` como variable de entorno `DB_USERNAME`.
- Monte todo el secret como un volumen en `/etc/db-secret`.

**Solución:**

1. Crear el Secret:
```bash
kubectl create secret generic db-secret --from-literal=db-user=admin --from-literal=db-pass=password123
```

2. Crear YAML del pod `secret-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: db-user
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/db-secret
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: db-secret
```
3. Aplicar:
```bash
kubectl apply -f secret-pod.yaml
```

**Verificación:**
```bash
kubectl exec secret-pod -- env | grep DB_USERNAME
kubectl exec secret-pod -- cat /etc/db-secret/db-pass
```
*Doc URL:* [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

---

## Ejercicio 7: Init Containers
**Tiempo estimado:** 5 minutos

**Enunciado:**
Crea un pod llamado `init-pod` con un contenedor principal basado en `busybox` que ejecute `sleep 3600`.
Añade un init container llamado `init-wait` basado en `busybox` que ejecute el comando `sh -c 'until nslookup kubernetes.default.svc.cluster.local; do echo waiting for k8s service; sleep 2; done'`.

**Solución:**

1. Crear YAML `init-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  initContainers:
  - name: init-wait
    image: busybox
    command: ['sh', '-c', 'until nslookup kubernetes.default.svc.cluster.local; do echo waiting for k8s service; sleep 2; done']
  containers:
  - name: main-container
    image: busybox
    command: ['sleep', '3600']
```
2. Aplicar:
```bash
kubectl apply -f init-pod.yaml
```

**Verificación:**
```bash
kubectl get pod init-pod
kubectl logs init-pod -c init-wait
```
*Doc URL:* [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

---

## Ejercicio 8: Multi-container Pod (sidecar)
**Tiempo estimado:** 6 minutos

**Enunciado:**
Crea un pod llamado `multi-pod` que contenga dos contenedores:
1. `app`: usa la imagen `busybox` y ejecuta `sh -c 'while true; do echo "Log entry" >> /var/log/app.log; sleep 5; done'`
2. `sidecar`: usa la imagen `busybox` y ejecuta `tail -f /var/log/app.log`
Ambos contenedores deben compartir un volumen `emptyDir` montado en `/var/log`.

**Solución:**

1. Crear YAML `multi-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  volumes:
  - name: log-vol
    emptyDir: {}
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Log entry" >> /var/log/app.log; sleep 5; done']
    volumeMounts:
    - name: log-vol
      mountPath: /var/log
  - name: sidecar
    image: busybox
    command: ['tail', '-f', '/var/log/app.log']
    volumeMounts:
    - name: log-vol
      mountPath: /var/log
```
2. Aplicar:
```bash
kubectl apply -f multi-pod.yaml
```

**Verificación:**
```bash
kubectl logs multi-pod -c sidecar
```
*Doc URL:* [Communicate Between Containers in the Same Pod](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)

---

## Ejercicio 9: Jobs y CronJobs
**Tiempo estimado:** 6 minutos

**Enunciado:**
1. Crea un Job llamado `pi-job` basado en la imagen `perl` que ejecute el comando `perl -Mbignum=bpi -wle 'print bpi(2000)'`.
2. Crea un CronJob llamado `ping-job` usando la imagen `busybox` que ejecute `ping -c 3 google.com` cada 5 minutos.

**Solución:**

1. Crear el Job imperativamente:
```bash
kubectl create job pi-job --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```
2. Crear el CronJob imperativamente:
```bash
kubectl create cronjob ping-job --image=busybox --schedule="*/5 * * * *" -- ping -c 3 google.com
```

**Verificación:**
```bash
kubectl get jobs
kubectl logs job/pi-job
kubectl get cronjobs
```
*Doc URL:* [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/), [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)

---

## Ejercicio 10: Node Affinity
**Tiempo estimado:** 5 minutos

**Enunciado:**
Asigna el label `disktype=ssd` a uno de tus nodos workers (ej. `node01`).
Crea un pod llamado `affinity-pod` con la imagen `nginx` que utilice `nodeAffinity` (requiredDuringSchedulingIgnoredDuringExecution) para programarse únicamente en nodos con la etiqueta `disktype=ssd`.

**Solución:**

1. Etiquetar el nodo (reemplaza `node01` por el nombre real de tu nodo):
```bash
kubectl label node node01 disktype=ssd
```

2. Crear YAML `affinity-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
```
3. Aplicar:
```bash
kubectl apply -f affinity-pod.yaml
```

**Verificación:**
```bash
kubectl get pod affinity-pod -o wide
```
El pod debe estar corriendo en el nodo etiquetado.
*Doc URL:* [Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

---

## Ejercicio 11: Taints y Tolerations
**Tiempo estimado:** 5 minutos

**Enunciado:**
Añade un taint al nodo `node01` (o a uno de tus nodos) con la llave `key1`, valor `value1` y el efecto `NoSchedule`.
Crea un pod llamado `tol-pod` con la imagen `nginx` que posea la toleration necesaria para poder ejecutarse en este nodo, e indícale explícitamente a través de `nodeName` que se programe allí.

**Solución:**

1. Aplicar el Taint al nodo:
```bash
kubectl taint nodes node01 key1=value1:NoSchedule
```

2. Crear YAML `tol-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tol-pod
spec:
  nodeName: node01
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```
3. Aplicar:
```bash
kubectl apply -f tol-pod.yaml
```

**Verificación:**
```bash
kubectl get pod tol-pod -o wide
# Opcional: remover el taint luego
kubectl taint nodes node01 key1=value1:NoSchedule-
```
*Doc URL:* [Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

---

## Ejercicio 12: Static Pods
**Tiempo estimado:** 4 minutos

**Enunciado:**
Crea un static pod llamado `static-web` utilizando la imagen `nginx` en el nodo de control plane (master).
El static pod debe estar gestionado por el kubelet del master.

**Solución:**

1. Encuentra la ruta de los static pods. Habitualmente es `/etc/kubernetes/manifests`. Si no estás seguro, revisa la configuración de kubelet:
```bash
cat /var/lib/kubelet/config.yaml | grep staticPodPath
```
2. Genera el manifiesto del pod en esa ruta:
```bash
kubectl run static-web --image=nginx --dry-run=client -o yaml > /etc/kubernetes/manifests/static-web.yaml
```
*Nota: Si estás haciendo el examen y no tienes acceso root por defecto, usa `sudo`.*

**Verificación:**
```bash
kubectl get pods -A | grep static-web
```
El nombre del pod tendrá el sufijo del nodo (ej. `static-web-controlplane`).
*Doc URL:* [Static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

---

## Ejercicio 13: DaemonSet
**Tiempo estimado:** 4 minutos

**Enunciado:**
Crea un DaemonSet llamado `fluentd-ds` en el namespace `kube-system` que corra la imagen `fluent/fluentd:v1.14-1`.

**Solución:**

No hay comando imperativo directo para DaemonSet. Crea un archivo `daemonset.yaml`:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-ds
  namespace: kube-system
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14-1
```
```bash
kubectl apply -f daemonset.yaml
```

**Verificación:**
```bash
kubectl get daemonset -n kube-system fluentd-ds
kubectl get pods -n kube-system -l app=fluentd -o wide
```
*Doc URL:* [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

---

## Ejercicio 14: Pod con múltiples contenedores compartiendo volumen
**Tiempo estimado:** 5 minutos

**Enunciado:**
*(Similar al Ejercicio 8, pero con enfoque en lectura/escritura de datos compartidos, un escenario muy común).*
Crea un pod llamado `share-pod` con dos contenedores.
Contenedor 1 (`writer`): imagen `busybox`, ejecuta un bucle que escriba la fecha cada 2 segundos en `/data/index.html`.
Contenedor 2 (`reader`): imagen `nginx`, monta el mismo volumen en `/usr/share/nginx/html`.
El volumen a usar debe ser de tipo `emptyDir`.

**Solución:**

1. Crear YAML `share-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: share-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  containers:
  - name: writer
    image: busybox
    command: ["/bin/sh", "-c"]
    args: ["while true; do date > /data/index.html; sleep 2; done"]
    volumeMounts:
    - name: shared-data
      mountPath: /data
  - name: reader
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
```
2. Aplicar:
```bash
kubectl apply -f share-pod.yaml
```

**Verificación:**
```bash
kubectl exec share-pod -c reader -- curl localhost
```
Debe imprimir la fecha actual.
*Doc URL:* [Communicate Between Containers in the Same Pod](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)
