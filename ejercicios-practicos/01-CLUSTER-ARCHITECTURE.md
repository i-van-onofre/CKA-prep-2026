# Dominio: Cluster Architecture, Installation & Configuration (25% del examen CKA)

A continuación se presentan 12 ejercicios prácticos estilo examen CKA. En el examen real dispondrás de aproximadamente 2 horas para 15-17 preguntas, por lo que la velocidad y el uso imperativo de `kubectl` son clave.

---

## Ejercicio 1: Gestionar RBAC (Role y RoleBinding)
**Tiempo estimado:** ~4 min
**Dificultad:** Media
**Documentación:** https://kubernetes.io/docs/reference/access-authn-authz/rbac/

**Enunciado:**
Crea un namespace llamado `web-team`. Dentro de este namespace, crea un `Role` llamado `deploy-manager` que permita listar (`list`) y crear (`create`) `deployments`. Luego, crea un `RoleBinding` llamado `deploy-manager-binding` que asigne este Role al usuario `developer`.

**Solución paso a paso:**
1. Crear el namespace:
```bash
kubectl create namespace web-team
```
2. Crear el Role imperativamente:
```bash
kubectl create role deploy-manager --verb=list,create --resource=deployments -n web-team
```
3. Crear el RoleBinding imperativamente:
```bash
kubectl create rolebinding deploy-manager-binding --role=deploy-manager --user=developer -n web-team
```

**Comando de verificación:**
```bash
kubectl auth can-i create deployments --as=developer -n web-team
# Debe devolver: yes
```

---

## Ejercicio 2: Gestionar RBAC ClusterRole
**Tiempo estimado:** ~5 min
**Dificultad:** Media
**Documentación:** https://kubernetes.io/docs/reference/access-authn-authz/rbac/

**Enunciado:**
Crea un `ServiceAccount` llamado `node-viewer` en el namespace `default`. Crea un `ClusterRole` llamado `node-pv-viewer` que permita listar (`list`) `nodes` y `persistentvolumes`. Vincula este ClusterRole al ServiceAccount creado usando un `ClusterRoleBinding` llamado `node-pv-viewer-binding`.

**Solución paso a paso:**
1. Crear el ServiceAccount:
```bash
kubectl create serviceaccount node-viewer -n default
```
2. Crear el ClusterRole:
```bash
kubectl create clusterrole node-pv-viewer --verb=list --resource=nodes,persistentvolumes
```
3. Crear el ClusterRoleBinding:
```bash
kubectl create clusterrolebinding node-pv-viewer-binding --clusterrole=node-pv-viewer --serviceaccount=default:node-viewer
```

**Comando de verificación:**
```bash
kubectl auth can-i list nodes --as=system:serviceaccount:default:node-viewer
# Debe devolver: yes
```

---

## Ejercicio 3: Usar kubeadm para instalar un cluster
**Tiempo estimado:** ~8 min
**Dificultad:** Alta
**Documentación:** https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

**Enunciado:**
Documenta los pasos y comandos necesarios para inicializar un control plane usando `kubeadm` definiendo el rango de red de los pods como `10.244.0.0/16`. Posteriormente, muestra el comando para unir un worker node al cluster.

**Solución paso a paso:**
1. Inicializar el control plane:
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
2. Configurar el acceso `kubectl` para el usuario actual (aparece en la salida de `kubeadm init`):
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
3. Instalar un plugin de red (ej. Flannel):
```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```
4. En el nodo Worker, ejecutar el comando de unión (generado por el init, o se puede regenerar):
```bash
# Para generar un nuevo token y comando de join desde el control plane:
kubeadm token create --print-join-command

# En el nodo worker:
sudo kubeadm join <control-plane-host>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

**Comando de verificación:**
```bash
kubectl get nodes
# Ambos nodos deben aparecer en estado "Ready"
```

---

## Ejercicio 4: Upgrade de cluster con kubeadm
**Tiempo estimado:** ~10 min
**Dificultad:** Alta
**Documentación:** https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

**Enunciado:**
Actualiza el nodo control-plane (asume que se llama `k8s-master`) de la versión v1.30.0 a la v1.31.0. Debes seguir el procedimiento seguro de actualización: drenar, actualizar kubeadm, actualizar componentes, actualizar kubelet/kubectl y volver a poner el nodo en servicio.

**Solución paso a paso:**
1. Actualizar `kubeadm` en el OS:
```bash
sudo apt-mark unhold kubeadm
sudo apt-get update && sudo apt-get install -y kubeadm=1.31.0-1.1
sudo apt-mark hold kubeadm
```
2. Drenar el nodo del control plane:
```bash
kubectl drain k8s-master --ignore-daemonsets
```
3. Planificar y aplicar la actualización de Kubernetes:
```bash
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.31.0
```
4. Actualizar `kubelet` y `kubectl`:
```bash
sudo apt-mark unhold kubelet kubectl
sudo apt-get update && sudo apt-get install -y kubelet=1.31.0-1.1 kubectl=1.31.0-1.1
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
5. Hacer el uncordon del nodo:
```bash
kubectl uncordon k8s-master
```

**Comando de verificación:**
```bash
kubectl get nodes
# El nodo debe mostrar la versión v1.31.0
```

---

## Ejercicio 5: Backup y Restore de etcd
**Tiempo estimado:** ~7 min
**Dificultad:** Alta
**Documentación:** https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

**Enunciado:**
Realiza un snapshot de la base de datos `etcd` y guárdalo en `/opt/backup/etcd-snapshot.db`. El endpoint de etcd es `https://127.0.0.1:2379`.
Posteriormente, restaura el snapshot en un nuevo directorio de datos `/var/lib/etcd-restore`.

**Solución paso a paso:**
1. Tomar el snapshot:
```bash
ETCDCTL_API=3 etcdctl snapshot save /opt/backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```
2. Restaurar el snapshot:
```bash
ETCDCTL_API=3 etcdctl snapshot restore /opt/backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore \
  --initial-cluster=master=https://127.0.0.1:2380 \
  --initial-cluster-token=etcd-cluster-1 \
  --name=master
```
*(Nota: En un escenario real tras la restauración, editarías el volumen `hostPath` en el manifiesto estático `/etc/kubernetes/manifests/etcd.yaml` para que apunte al nuevo directorio `/var/lib/etcd-restore`).*

**Comando de verificación:**
```bash
ETCDCTL_API=3 etcdctl snapshot status /opt/backup/etcd-snapshot.db -w table
```

---

## Ejercicio 6: Gestionar certificados TLS
**Tiempo estimado:** ~4 min
**Dificultad:** Media
**Documentación:** https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/

**Enunciado:**
Verifica la fecha de expiración de los certificados del control plane gestionados por kubeadm. Luego, ejecuta el comando para renovar manualmente todos los certificados.

**Solución paso a paso:**
1. Verificar expiración:
```bash
sudo kubeadm certs check-expiration
```
2. Renovar todos los certificados:
```bash
sudo kubeadm certs renew all
```
3. Reiniciar el control plane para que tome los nuevos certificados (por ejemplo, etcd, kube-apiserver, kube-controller-manager, kube-scheduler). La forma más directa en un entorno local es matar los contenedores, kubelet los recreará:
```bash
sudo crictl ps | grep kube-apiserver | awk '{print $1}' | xargs sudo crictl stop
# (Repetir para scheduler y controller-manager)
```

**Comando de verificación:**
```bash
sudo kubeadm certs check-expiration
# Deberías ver que la fecha de caducidad se ha extendido a un año a partir de hoy.
```

---

## Ejercicio 7: Configurar ServiceAccount y verificar permisos
**Tiempo estimado:** ~4 min
**Dificultad:** Baja
**Documentación:** https://kubernetes.io/docs/reference/access-authn-authz/rbac/#service-account-permissions

**Enunciado:**
Crea un ServiceAccount llamado `pipeline-sa` en el namespace `ci-cd`. Asígnale el ClusterRole `edit` limitando su alcance solo al namespace `ci-cd`. Verifica que este ServiceAccount puede crear pods en `ci-cd` pero no en `default`.

**Solución paso a paso:**
1. Crear el namespace y el ServiceAccount:
```bash
kubectl create namespace ci-cd
kubectl create serviceaccount pipeline-sa -n ci-cd
```
2. Crear un RoleBinding que asigne un ClusterRole (`edit`) dentro de un namespace:
```bash
kubectl create rolebinding pipeline-sa-binding \
  --clusterrole=edit \
  --serviceaccount=ci-cd:pipeline-sa \
  -n ci-cd
```

**Comando de verificación:**
```bash
kubectl auth can-i create pods --as=system:serviceaccount:ci-cd:pipeline-sa -n ci-cd
# yes
kubectl auth can-i create pods --as=system:serviceaccount:ci-cd:pipeline-sa -n default
# no
```

---

## Ejercicio 8: NetworkPolicy (Ingress)
**Tiempo estimado:** ~6 min
**Dificultad:** Media
**Documentación:** https://kubernetes.io/docs/concepts/services-networking/network-policies/

**Enunciado:**
En el namespace `app-space`, crea una `NetworkPolicy` llamada `backend-policy`. Esta política debe permitir el tráfico Ingress entrante al puerto TCP 80 de los pods con el label `app=backend` ÚNICAMENTE desde los pods que tengan el label `app=frontend`. Deniega el resto del tráfico Ingress hacia el backend.

**Solución paso a paso:**
1. Crear el manifiesto YAML `np-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: app-space
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```
2. Aplicar la política:
```bash
kubectl create ns app-space
kubectl apply -f np-ingress.yaml
```

**Comando de verificación:**
```bash
kubectl describe networkpolicy backend-policy -n app-space
```

---

## Ejercicio 9: NetworkPolicy (Egress)
**Tiempo estimado:** ~5 min
**Dificultad:** Media
**Documentación:** https://kubernetes.io/docs/concepts/services-networking/network-policies/

**Enunciado:**
Crea una `NetworkPolicy` llamada `restrict-egress` en el namespace `secure-ns` que bloquee TODO el tráfico de salida (Egress) de todos los pods en dicho namespace, EXCEPTO el tráfico de resolución DNS (puerto 53 UDP/TCP).

**Solución paso a paso:**
1. Crear el archivo `np-egress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress
  namespace: secure-ns
spec:
  podSelector: {} # Selecciona todos los pods del namespace
  policyTypes:
  - Egress
  egress:
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```
2. Aplicar:
```bash
kubectl create ns secure-ns
kubectl apply -f np-egress.yaml
```

**Comando de verificación:**
```bash
kubectl describe networkpolicy restrict-egress -n secure-ns
```

---

## Ejercicio 10: Drenar un nodo para mantenimiento
**Tiempo estimado:** ~3 min
**Dificultad:** Baja
**Documentación:** https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

**Enunciado:**
El nodo `worker-02` necesita mantenimiento de hardware. Prepara el nodo para que no acepte nuevos pods y desaloja los existentes (ignorando DaemonSets y eliminando datos locales si los hubiera). Una vez completado el "mantenimiento", vuelve a hacerlo elegible para programar pods.

**Solución paso a paso:**
1. Marcar el nodo como unschedulable (Cordon) y evacuar los pods (Drain):
```bash
kubectl drain worker-02 --ignore-daemonsets --delete-emptydir-data --force
```
*(Nota: El comando `drain` automáticamente aplica un `cordon` al nodo).*

2. Devolver el nodo a su estado normal (Uncordon):
```bash
kubectl uncordon worker-02
```

**Comando de verificación:**
```bash
kubectl get nodes worker-02
# SchedulingDisabled no debe aparecer bajo STATUS.
```

---

## Ejercicio 11: Crear usuario con certificado (CSR)
**Tiempo estimado:** ~8 min
**Dificultad:** Alta
**Documentación:** https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user

**Enunciado:**
Tienes una clave privada de usuario generada: `johndoe.key` y su Certificate Signing Request: `johndoe.csr`.
Crea un objeto `CertificateSigningRequest` en Kubernetes para el usuario `johndoe`, apruébalo y extrae el certificado firmado. Finalmente, añade al usuario al kubeconfig.
*(A efectos prácticos, generaremos la clave y el csr primero).*

**Solución paso a paso:**
1. Generar key y CSR:
```bash
openssl genrsa -out johndoe.key 2048
openssl req -new -key johndoe.key -out johndoe.csr -subj "/CN=johndoe/O=developers"
```
2. Crear el YAML del CSR codificando en base64:
```bash
export BASE64_CSR=$(cat johndoe.csr | base64 | tr -d '\n')
cat <<EOF > csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: johndoe-csr
spec:
  request: $BASE64_CSR
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
kubectl apply -f csr.yaml
```
3. Aprobar el CSR:
```bash
kubectl certificate approve johndoe-csr
```
4. Extraer el certificado firmado:
```bash
kubectl get csr johndoe-csr -o jsonpath='{.status.certificate}' | base64 --decode > johndoe.crt
```
5. Configurar kubeconfig:
```bash
kubectl config set-credentials johndoe --client-certificate=johndoe.crt --client-key=johndoe.key
```

**Comando de verificación:**
```bash
kubectl config view
# Debe aparecer el usuario johndoe con las rutas a su crt y key.
```

---

## Ejercicio 12: Gestionar contextos kubeconfig
**Tiempo estimado:** ~3 min
**Dificultad:** Baja
**Documentación:** https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

**Enunciado:**
Crea un nuevo contexto en tu `kubeconfig` llamado `dev-context`. Este contexto debe utilizar el cluster `kubernetes`, el usuario `johndoe` (creado en el ejercicio anterior) y el namespace predeterminado `dev-env`. Finalmente, cambia tu contexto activo a `dev-context`.

**Solución paso a paso:**
1. Crear el nuevo contexto:
```bash
kubectl config set-context dev-context \
  --cluster=kubernetes \
  --user=johndoe \
  --namespace=dev-env
```
2. Cambiar al nuevo contexto:
```bash
kubectl config use-context dev-context
```

**Comando de verificación:**
```bash
kubectl config current-context
# Debe mostrar: dev-context
```
