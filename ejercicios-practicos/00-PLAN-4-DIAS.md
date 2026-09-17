# 🏆 Plan Intensivo de Estudio CKA (4 Días)

Este plan está diseñado para optimizar tu preparación final antes del examen **Certified Kubernetes Administrator (CKA)**.

---

## 📅 1. Plan de Estudio (4 Días)

### Día 1: Cluster Architecture, Installation & Configuration (25%)
**Objetivo:** Dominar la infraestructura base y los componentes del clúster.
- [ ] Gestión de acceso basado en roles (RBAC) y Service Accounts.
- [ ] Uso de Kubeadm para instalar un clúster básico y añadir nodos.
- [ ] Gestión de un clúster de alta disponibilidad.
- [ ] Realizar actualizaciones de versión en un clúster Kubernetes usando Kubeadm (Upgrade de Control Plane y Worker Nodes).
- [ ] Implementar y gestionar copias de seguridad de etcd (Backup & Restore usando `etcdctl`).

### Día 2: Workloads & Scheduling (15%) + Services & Networking (20%)
**Objetivo:** Despliegue de aplicaciones, asignación de recursos y conectividad del clúster.
- [ ] Entender y gestionar Deployments, rolling updates y rollbacks.
- [ ] Configurar ConfigMaps y Secrets para inyectarlos en Pods como variables de entorno o volúmenes.
- [ ] Conocer los primitivos de planificación de Pods (Node Selectors, Node Affinity, Pod Affinity/Anti-Affinity, Taints & Tolerations).
- [ ] Entender Services (ClusterIP, NodePort, LoadBalancer) y cómo mapean a Endpoints.
- [ ] Desplegar Ingress Controllers y crear reglas de Ingress para enrutamiento HTTP/HTTPS.
- [ ] Configurar Network Policies para restringir o permitir el tráfico a nivel de Pod.
- [ ] Uso de CoreDNS y entender cómo funciona la resolución de nombres DNS en el clúster.

### Día 3: Storage (10%) + Troubleshooting (30%)
**Objetivo:** Persistencia de datos y resolución rápida de problemas críticos.
- [ ] Entender Storage Classes (SC), Persistent Volumes (PV) y Persistent Volume Claims (PVC).
- [ ] Configurar volúmenes con modos de acceso apropiados (`ReadWriteOnce`, `ReadOnlyMany`, etc.).
- [ ] Evaluar logs a nivel de clúster y de nodo usando `kubectl logs` y `journalctl`.
- [ ] Solucionar problemas con componentes del plano de control (kube-apiserver, kube-scheduler, kube-controller-manager).
- [ ] Solucionar problemas en nodos trabajadores (kubelet, container runtime) y estados de Pods (CrashLoopBackOff, Pending, ImagePullBackOff).
- [ ] Diagnosticar problemas de red (Services sin Endpoints, NetworkPolicies mal configuradas, problemas de CNI).

### Día 4: Simulacros de Examen y Repaso Final
**Objetivo:** Poner todo en práctica bajo presión de tiempo simulando el entorno real.
- [ ] Realizar simulacros completos (e.g., Killer.sh).
- [ ] Practicar la generación de YAML imperativa sin mirar la documentación.
- [ ] Repasar los errores cometidos en los simulacros y buscar por qué fallaste.
- [ ] Practicar navegación rápida por la documentación oficial.

---

## 💡 2. Tips Esenciales para el Examen CKA

### 🎯 Pesos del Examen (Dominios Actualizados)
- **Troubleshooting:** 30%
- **Cluster Architecture, Installation & Configuration:** 25%
- **Services & Networking:** 20%
- **Workloads & Scheduling:** 15%
- **Storage:** 10%

### ⏱ Gestión del Tiempo
- **Duración:** 120 minutos (2 horas).
- **Preguntas:** Aproximadamente 17 preguntas.
- **Tiempo por pregunta:** ~7 minutos en promedio.
- **Tip de Oro:** Si una pregunta te toma más de 7 minutos o vale muy pocos puntos (2-4%), **márcala (Flag)** y sáltala. Vuelve a ella al final si sobra tiempo.

### ⚙️ Configuración del Entorno de Bash (Primeros 2 minutos)
Al iniciar tu entorno de examen, abre la terminal y configura bash para ser ultra rápido:

```bash
# Autocompletado (a menudo ya configurado, pero no asumas nada)
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

# Alias esencial (te ahorrará miles de pulsaciones)
alias k=kubectl
complete -o default -F __start_kubectl k

# Variables para generar YAML rápido de prueba
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0" # Para borrar pods instantáneamente
```
*Ejemplo de uso:* `k run nginx --image=nginx $do > pod.yaml`

### ⚡ Generación Rápida de YAML (Comandos Imperativos)
En el examen NO hay tiempo para escribir YAML desde cero. Genera las plantillas usando la CLI y luego edítalas con `vim` o `nano`.
- **Pod:** `k run pod-name --image=nginx $do > pod.yaml`
- **Deployment:** `k create deploy my-deploy --image=nginx --replicas=3 $do > deploy.yaml`
- **Service:** `k expose pod my-pod --port=80 --name=my-svc $do > svc.yaml`
- **Secret/ConfigMap:** `k create secret generic my-secret --from-literal=key=value $do > secret.yaml`

### 🔄 Cambio de Contexto (¡CRÍTICO Y FATAL SI LO OLVIDAS!)
El examen tiene múltiples clústeres. Antes de responder CADA pregunta, **SIEMPRE** ejecuta el comando proporcionado en el encabezado de la pregunta para estar en el clúster correcto:
```bash
kubectl config use-context <nombre-contexto>
```

### 📚 Uso de la Documentación Oficial (Open Book)
Puedes acceder a la documentación oficial: `https://kubernetes.io/docs/`.
- Usa la barra de búsqueda eficientemente. No intentes leer, busca conceptos clave y ve directo a los fragmentos de YAML.
- **Términos Clave de Búsqueda (Bookmarks mentales):**
  - *Kubeadm upgrade* (Página: Upgrading kubeadm clusters)
  - *etcd backup* (Página: Operating etcd clusters for Kubernetes)
  - *Network Policy* (Copia los ejemplos de Allow/Deny traffic)
  - *Persistent Volume* (Plantillas para hostPath o NFS)
  - *Ingress* (Ejemplo de Ingress Resource básico)

---

## ✅ 3. Checklist de Temas que DEBES Dominar (Marcar antes del examen)

- [ ] Realizar un upgrade completo de Control Plane y Node usando `kubeadm upgrade`.
- [ ] Hacer backup de la DB etcd usando `etcdctl snapshot save` y restaurarlo con `etcdctl snapshot restore`.
- [ ] Filtrar salidas de kubectl usando JSONPath (ej. `k get nodes -o jsonpath="{.items[*].metadata.name}"`).
- [ ] Crear un NetworkPolicy para aislar un namespace completo o permitir solo tráfico en el puerto 80.
- [ ] Encontrar un nodo problemático usando `kubectl top nodes` / `kubectl top pods`.
- [ ] Diagnosticar un nodo `NotReady` reiniciando el servicio local (`systemctl status kubelet`, `journalctl -u kubelet`).
- [ ] Agregar Taints a un nodo (`k taint nodes node-name key=value:NoSchedule`) y configurar su Toleration respectiva en el YAML del Pod.
- [ ] Crear y vincular un PersistentVolume a un PersistentVolumeClaim.
- [ ] Desplegar un Ingress, especificando el host, el path (`/`), el service backend y su puerto.
- [ ] Diferenciar y crear ClusterRole, ClusterRoleBinding, Role, RoleBinding y ServiceAccounts.

---

## 🚀 4. Recursos Recomendados

1. **Killer.sh**: Recibes 2 sesiones del simulador gratuito al comprar el examen de la Linux Foundation. Las preguntas son más difíciles que el examen real, si logras pasar killer.sh, aprobarás el CKA fácilmente.
2. **kubernetes.io/docs**: Acostúmbrate a su estructura y a buscar rápidamente. En especial, familiarízate con la sección "Cheat Sheet".
3. **Curso de Mumshad Mannambeth (Udemy o KodeKloud)**: "Kubernetes Certified Administrator (CKA) with Practice Tests". Es el mejor curso del mercado con laboratorios prácticos en vivo incluidos.
4. **Repositorio kubernetes/kubectl**: Puedes visitar github.com/kubernetes en el examen, útil si te sabes rutas a ciertos YAMLs, pero la doc oficial debería ser más que suficiente.
