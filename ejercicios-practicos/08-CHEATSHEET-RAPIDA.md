# CKA - Cheatsheet de Referencia Rápida

## 1. Setup Inicial (HACER PRIMERO EN EL EXAMEN)

Ejecuta esto en cada terminal nueva para ganar velocidad:

```bash
alias k=kubectl
export do='--dry-run=client -o yaml'
export now='--force --grace-period=0'
source <(kubectl completion bash)
complete -o default -F __start_kubectl k
```

## 2. Generación Rápida de YAML (Imperativo)

Usa siempre el flag `$do` para generar el YAML base y redirígelo a un archivo (`> file.yaml`).

- **Pod:** `k run nginx --image=nginx $do > pod.yaml`
- **Deployment:** `k create deployment nginx --image=nginx --replicas=3 $do > deploy.yaml`
- **Service ClusterIP:** `k expose deploy nginx --port=80 --target-port=80 $do`
- **Service NodePort:** `k expose deploy nginx --type=NodePort --port=80 $do`
- **Job:** `k create job test --image=busybox -- /bin/sh -c 'echo hello'`
- **CronJob:** `k create cronjob test --image=busybox --schedule='*/5 * * * *' -- /bin/sh -c 'echo hello'`
- **ConfigMap:** `k create configmap my-cm --from-literal=key1=value1`
- **Secret:** `k create secret generic my-secret --from-literal=password=admin`
- **ServiceAccount:** `k create sa my-sa`
- **Role:** `k create role pod-reader --verb=get,list --resource=pods`
- **RoleBinding:** `k create rolebinding pod-reader-binding --role=pod-reader --user=john`
- **ClusterRole:** `k create clusterrole node-reader --verb=get,list --resource=nodes`
- **ClusterRoleBinding:** `k create clusterrolebinding node-reader-binding --clusterrole=node-reader --user=admin`
- **Ingress:** `k create ingress my-ingress --rule='host/path=svc:port'`
- **Namespace:** `k create ns my-namespace`

## 3. Comandos de Diagnóstico Esenciales

- **Ver todos los pods:** `k get pods -A`
- **Detalles de un pod (eventos al final):** `k describe pod <name>`
- **Ver logs (actual):** `k logs <pod> [-c container]`
- **Ver logs (previo al crash):** `k logs <pod> --previous`
- **Entrar al contenedor:** `k exec -it <pod> -- /bin/sh`
- **Ver eventos del cluster (ordenados):** `k get events --sort-by='.lastTimestamp'`
- **Consumo de recursos:** `k top pods` / `k top nodes`
- **Verificar permisos (RBAC):** `k auth can-i <verb> <resource> --as <user>`
- **Contenedores a nivel nodo (cuando kubectl falla):** `crictl ps` / `crictl logs <container-id>`
- **Estado de kubelet:** `systemctl status kubelet`
- **Logs de kubelet:** `journalctl -u kubelet`

## 4. Templates YAML Esenciales

Copia y pega desde la documentación o ten en mente estas estructuras básicas.

### Pod (Básico, Resources, Volume)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    volumeMounts:
    - name: data-vol
      mountPath: /usr/share/data
  volumes:
  - name: data-vol
    emptyDir: {}
```

### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx
```

### PV (hostPath) & PVC
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
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
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### NetworkPolicy (Default Deny Ingress & Allow Specific)
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 3306
```

### Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: my-app.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-svc
            port:
              number: 80
```

## 5. etcd Backup y Restore

Usa siempre el certificado, la key y la CA correctas. Asegúrate de estar en el nodo master o apuntar a él.

**Backup:**
```bash
ETCDCTL_API=3 etcdctl snapshot save /path/to/backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

**Restore:** (Normalmente se restaura a un nuevo directorio y se edita el manifiesto estático del etcd)
```bash
ETCDCTL_API=3 etcdctl snapshot restore /path/to/backup.db \
  --data-dir=/var/lib/etcd-backup
```
*Luego editar `/etc/kubernetes/manifests/etcd.yaml` y cambiar los `hostPath` al nuevo directorio.*

## 6. Cluster Upgrade con kubeadm

**En el Control Plane:**
1. `apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.x.x-00 && apt-mark hold kubeadm`
2. `kubeadm upgrade plan`
3. `kubeadm upgrade apply v1.x.x`
4. Drenar nodo: `kubectl drain <node-name> --ignore-daemonsets`
5. `apt-mark unhold kubelet kubectl && apt-get install -y kubelet=1.x.x-00 kubectl=1.x.x-00 && apt-mark hold kubelet kubectl`
6. `systemctl daemon-reload && systemctl restart kubelet`
7. `kubectl uncordon <node-name>`

**En el Worker Node:**
1. Drenar nodo desde el master: `kubectl drain <node-name> --ignore-daemonsets`
2. SSH al worker: `apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.x.x-00 && apt-mark hold kubeadm`
3. `kubeadm upgrade node`
4. `apt-mark unhold kubelet kubectl && apt-get install -y kubelet=1.x.x-00 kubectl=1.x.x-00 && apt-mark hold kubelet kubectl`
5. `systemctl daemon-reload && systemctl restart kubelet`
6. Desdrenar desde el master: `kubectl uncordon <node-name>`

## 7. URLs de Documentación Clave

Ten estos términos listos para buscar en kubernetes.io/docs:
- **"kubectl cheat sheet"** (atajos y comandos)
- **"pv hostpath"** (ejemplo de PersistentVolume)
- **"network policy"** (ejemplos de Ingress/Egress)
- **"ingress"** (ejemplo de Ingress Resource)
- **"kubeadm upgrade"** (pasos de actualización)
- **"etcd snapshot"** (backup y restore)
- **"emptyDir"** (volúmenes temporales)
- **"init container"** (ejemplos de initContainers)

## 8. Trucos de Velocidad

- **Ver labels:** `k get po --show-labels`
- **Filtrar por label:** `k get po -l app=nginx`
- **Ver IP y nodo:** `k get po -o wide`
- **Ordenar por reinicios:** `k get po --sort-by='.status.containerStatuses[0].restartCount'`
- **Obtener solo los nombres:** `k get po -o jsonpath='{.items[*].metadata.name}'`
- **Recrear rápido un recurso:** `k replace --force -f file.yaml` (Usa `$now` para pods atascados en Terminating)
- **Pod temporal para debug (shell):** `k run tmp --rm -it --image=busybox -- sh`
- **Pod temporal para testear red (curl):** `k run tmp --rm -it --image=nginx:alpine -- curl svc-name.namespace.svc.cluster.local`
