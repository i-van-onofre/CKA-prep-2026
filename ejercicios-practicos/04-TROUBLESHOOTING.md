# Dominio: Troubleshooting (30% del examen CKA)

¡ATENCIÓN! Este es el dominio con mayor peso en el examen CKA. Debes ser rápido diagnosticando por qué un componente o recurso no funciona.

## Tips y Flujos de Diagnóstico para el CKA

1. **Pods fallando:**
   - `kubectl describe pod <nombre>` -> Revisa la sección "Events" (Eventos).
   - `kubectl logs <nombre>` -> Revisa la salida estándar del contenedor.
   - `kubectl logs <nombre> --previous` -> Revisa logs si el pod se reinició.
2. **Nodos fallando:**
   - Conéctate al nodo con `ssh <nodo>`.
   - `sudo systemctl status kubelet` -> Comprueba el servicio.
   - `sudo journalctl -u kubelet -f` -> Revisa los logs del kubelet.
3. **Componentes del Control Plane (Static Pods):**
   - Revisa `/etc/kubernetes/manifests/` para encontrar fallos de YAML.
   - Revisa los logs estáticos en `/var/log/pods/` o usa `crictl ps` y `crictl logs <container-id>` en el master node.

---

## Ejercicio 1: Pod en CrashLoopBackOff

**Tiempo estimado:** 4 minutos

**Escenario:**
Existe un pod llamado `app-backend` en el namespace `production` que está en estado `CrashLoopBackOff`. Diagnostica el problema y arréglalo para que el pod esté en estado `Running`.

**Solución paso a paso:**
1. Identificar el estado del pod:
   ```bash
   kubectl get pods -n production
   ```
2. Obtener los logs para ver por qué falla la aplicación:
   ```bash
   kubectl logs app-backend -n production
   ```
   *(Salida imaginaria: `sh: can't open 'strart.sh': No such file or directory`)*
3. Revisar la configuración del pod:
   ```bash
   kubectl get pod app-backend -n production -o yaml > fix-pod.yaml
   ```
4. Editar `fix-pod.yaml` y arreglar el error tipográfico en la sección `command` o `args` (ej. cambiar `strart.sh` a `start.sh`).
5. Recrear el pod:
   ```bash
   kubectl delete pod app-backend -n production --force --grace-period=0
   kubectl apply -f fix-pod.yaml
   ```

**Causa Raíz:** El comando de inicio definido en las especificaciones del contenedor estaba mal escrito, causando que el proceso saliera con error inmediatamente y entrara en un ciclo de reinicios (CrashLoopBackOff).

---

## Ejercicio 2: Pod en Pending

**Tiempo estimado:** 3 minutos

**Escenario:**
El pod `data-processor` en el namespace `default` se queda atascado en estado `Pending`. Encuentra la causa y soluciónalo.

**Solución paso a paso:**
1. Describir el pod para ver los eventos del scheduler:
   ```bash
   kubectl describe pod data-processor
   ```
   *(Salida en Events: `0/2 nodes are available: 2 Insufficient cpu.`)*
2. Exportar el pod para corregir el request:
   ```bash
   kubectl get pod data-processor -o yaml > pod-pending.yaml
   ```
3. Editar el archivo `pod-pending.yaml` y reducir la sección `resources.requests.cpu` a un valor que el nodo pueda satisfacer (por ejemplo, de `4000m` a `500m`).
4. Recrear el pod:
   ```bash
   kubectl replace --force -f pod-pending.yaml
   ```

**Causa Raíz:** Las peticiones de recursos (CPU/Memoria) del pod exceden la capacidad disponible en todos los nodos del clúster, por lo que el `kube-scheduler` no puede asignarlo a ningún nodo.

---

## Ejercicio 3: Pod en ImagePullBackOff

**Tiempo estimado:** 2 minutos

**Escenario:**
El pod `nginx-frontend` muestra el estado `ImagePullBackOff` o `ErrImagePull`.

**Solución paso a paso:**
1. Revisar los eventos:
   ```bash
   kubectl describe pod nginx-frontend
   ```
   *(Salida: `Failed to pull image "nginx:1.199.0": rpc error: code = NotFound...`)*
2. Modificar la imagen en vivo (o editar el YAML):
   ```bash
   kubectl set image pod/nginx-frontend nginx=nginx:1.19.0
   ```
3. Verificar:
   ```bash
   kubectl get pod nginx-frontend
   ```

**Causa Raíz:** La etiqueta (tag) de la imagen especificada en el YAML no existe en el registro de imágenes.

---

## Ejercicio 4: Deployment no escala

**Tiempo estimado:** 5 minutos

**Escenario:**
El deployment `web-app` tiene configuradas 3 réplicas, pero `kubectl get pods` solo muestra 1 (o ninguna) y no se crean nuevas, a pesar de usar `kubectl scale`. 

**Solución paso a paso:**
1. Revisar el estado del deployment y su ReplicaSet asociado:
   ```bash
   kubectl describe deployment web-app
   kubectl get rs
   kubectl describe rs <nombre-del-rs>
   ```
2. Buscar en los eventos del ReplicaSet por qué falla al crear los pods. A menudo es un error de formato YAML en la sección `template` del pod (ej: volumen inexistente, typo en un field).
3. Editar el deployment para corregir el error:
   ```bash
   kubectl edit deployment web-app
   ```
4. Verificar que los nuevos pods se crean correctamente.

**Causa Raíz:** El controlador del ReplicaSet no puede instanciar los pods debido a una validación fallida en las especificaciones del template del pod.

---

## Ejercicio 5: Service no conecta

**Tiempo estimado:** 4 minutos

**Escenario:**
El Service `db-svc` no envía tráfico a los pods `db-backend`.

**Solución paso a paso:**
1. Comprobar los Endpoints del Service:
   ```bash
   kubectl get endpoints db-svc
   ```
   *(Salida: `db-svc   <none>`)*
2. Comparar el selector del Service con los labels de los pods:
   ```bash
   kubectl get svc db-svc -o wide
   kubectl get pods --show-labels
   ```
3. Editar el servicio para que el `selector` coincida exactamente con las etiquetas del pod (ej. `app: db-backend` en lugar de `app: db-backnd`):
   ```bash
   kubectl edit svc db-svc
   ```
4. Validar:
   ```bash
   kubectl get endpoints db-svc
   ```

**Causa Raíz:** El `selector` del Service no coincide con los `labels` definidos en los pods, por lo que el objeto Endpoints queda vacío.

---

## Ejercicio 6: Nodo en NotReady

**Tiempo estimado:** 7 minutos

**Escenario:**
El nodo `worker-node-01` aparece como `NotReady`. Arregla el nodo.

**Solución paso a paso:**
1. Identificar el estado:
   ```bash
   kubectl get nodes
   ```
2. Conectarse al nodo afectado:
   ```bash
   ssh worker-node-01
   ```
3. Comprobar el servicio kubelet:
   ```bash
   sudo systemctl status kubelet
   ```
4. Si está detenido o fallando, revisar los logs:
   ```bash
   sudo journalctl -u kubelet -n 50 --no-pager
   ```
5. Si el problema es solo que está apagado:
   ```bash
   sudo systemctl start kubelet
   sudo systemctl enable kubelet
   ```
6. Desconectarse y verificar en el master:
   ```bash
   kubectl get nodes
   ```

**Causa Raíz:** El proceso kubelet, responsable de registrar el nodo con el API Server y gestionar contenedores localmente, estaba detenido o había crasheado.

---

## Ejercicio 7: kubelet no inicia por configuración incorrecta

**Tiempo estimado:** 8 minutos

**Escenario:**
Has reiniciado el `kubelet` pero falla inmediatamente.

**Solución paso a paso:**
1. Revisar los logs detallados del kubelet:
   ```bash
   sudo journalctl -u kubelet -f
   ```
   *(Salida en log: `failed to parse kubelet config file /var/lib/kubelet/config.yaml... unknown field`)*
2. Revisar el archivo de configuración mencionado:
   ```bash
   sudo vi /var/lib/kubelet/config.yaml
   ```
3. Buscar errores tipográficos en el YAML de configuración del Kubelet (ejemplo: puerto fuera de rango, variable mal tabulada) y corregirlo.
4. Reiniciar el servicio:
   ```bash
   sudo systemctl restart kubelet
   sudo systemctl status kubelet
   ```

**Causa Raíz:** Un archivo de configuración de Kubelet mal formado (`/var/lib/kubelet/config.yaml`) impide que el demonio se inicie.

---

## Ejercicio 8: Control plane caído (API Server no funciona)

**Tiempo estimado:** 8 minutos

**Escenario:**
Los comandos `kubectl` devuelven `The connection to the server <ip>:6443 was refused`. Recupera el clúster.

**Solución paso a paso:**
1. Entrar por SSH al Control Plane (master node).
2. Revisar los contenedores del master con `crictl`:
   ```bash
   sudo crictl ps -a | grep kube-apiserver
   ```
3. Inspeccionar logs del contenedor que falla:
   ```bash
   sudo crictl logs <container-id>
   ```
4. Revisar el Static Pod manifest del API server:
   ```bash
   sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml
   ```
5. Corregir cualquier parámetro erróneo (ejemplo: un typo en la ruta de los certificados o en una bandera como `--authorization-mode=RBAC` escrito mal como `RBACC`).
6. El kubelet detectará el cambio y reiniciará el pod automáticamente. Espera 1 minuto y prueba `kubectl get nodes`.

**Causa Raíz:** Un error en la configuración del manifiesto del static pod del kube-apiserver en `/etc/kubernetes/manifests/` causa que el contenedor falle al arrancar.

---

## Ejercicio 9: DNS no funciona

**Tiempo estimado:** 5 minutos

**Escenario:**
Los pods no pueden resolver nombres de servicios, pero pueden conectarse a IPs directas.

**Solución paso a paso:**
1. Revisar los pods de CoreDNS:
   ```bash
   kubectl get pods -n kube-system -l k8s-app=kube-dns
   ```
2. Si están en estado `CrashLoopBackOff`, revisa sus logs:
   ```bash
   kubectl logs -n kube-system -l k8s-app=kube-dns
   ```
3. Revisa la configuración del Corefile si el problema persiste:
   ```bash
   kubectl get configmap coredns -n kube-system -o yaml
   ```
4. Si hay un error, edítalo (`kubectl edit cm coredns -n kube-system`) y reinicia los pods:
   ```bash
   kubectl rollout restart deployment coredns -n kube-system
   ```

**Causa Raíz:** CoreDNS está fallando, a menudo por errores en su ConfigMap asociado o falta de recursos.

---

## Ejercicio 10: PVC en Pending

**Tiempo estimado:** 4 minutos

**Escenario:**
Un PersistentVolumeClaim (PVC) llamado `mysql-data` está en estado `Pending`.

**Solución paso a paso:**
1. Describir el PVC:
   ```bash
   kubectl describe pvc mysql-data
   ```
   *(Salida: `Failed to provision volume... no StorageClass` o `Cannot bind to requested volume`)*
2. Comprobar si existen PersistentVolumes (PV) libres:
   ```bash
   kubectl get pv
   ```
3. Si la StorageClass requerida no existe, crear el PV manualmente o corregir la StorageClass. Asegurarse que el `accessModes` y la capacidad requerida por el PVC coincidan o sean menores a las que ofrece el PV disponible.
4. Si hay un PV que puede usarse, editar el PVC o el PV para asegurar que no tengan `claimRef` residuales o mismatch de clases.

**Causa Raíz:** Falta de un PV disponible que coincida en capacidad, StorageClass y AccessModes con el PVC solicitado.

---

## Ejercicio 11: Pod no puede comunicarse entre namespaces

**Tiempo estimado:** 5 minutos

**Escenario:**
El pod `frontend` en el namespace `web` recibe timeouts al intentar contactar al `backend` en el namespace `api`.

**Solución paso a paso:**
1. Comprobar las NetworkPolicies existentes:
   ```bash
   kubectl get networkpolicy -n api
   ```
2. Describir la política de red aplicada al backend:
   ```bash
   kubectl describe networkpolicy <nombre-de-la-política> -n api
   ```
3. Verificar si el tráfico desde el namespace `web` está permitido.
4. Editar la política para permitir tráfico agregando el selector de namespace apropiado (ej: `namespaceSelector: matchLabels: name: web`):
   ```bash
   kubectl edit networkpolicy <nombre-de-la-política> -n api
   ```

**Causa Raíz:** Una política de red (NetworkPolicy) de tipo Ingress estaba bloqueando el tráfico no explícitamente permitido hacia los pods destino.

---

## Ejercicio 12: Certificados expirados

**Tiempo estimado:** 5 minutos

**Escenario:**
El clúster Kubeadm falla repentinamente y los logs muestran `x509: certificate has expired or is not yet valid`.

**Solución paso a paso:**
1. Comprobar la expiración de los certificados del clúster desde el nodo master:
   ```bash
   sudo kubeadm certs check-expiration
   ```
2. Renovar todos los certificados:
   ```bash
   sudo kubeadm certs renew all
   ```
3. Reiniciar los componentes del control plane (el método más rápido es mover los estáticos):
   ```bash
   sudo mv /etc/kubernetes/manifests/*.yaml /tmp/
   # Esperar a que los contenedores mueran, luego moverlos de vuelta:
   sudo mv /tmp/*.yaml /etc/kubernetes/manifests/
   ```
4. Actualizar la configuración kubeconfig local:
   ```bash
   sudo cp /etc/kubernetes/admin.conf ~/.kube/config
   ```

**Causa Raíz:** Los certificados generados por kubeadm tienen una validez por defecto de 1 año y deben ser rotados manualmente o mediante upgrades.

---

## Ejercicio 13: etcd no funciona

**Tiempo estimado:** 8 minutos

**Escenario:**
El control plane es inestable. Revisando los pods de kube-system, `etcd-master` está reiniciándose.

**Solución paso a paso:**
1. Obtener logs del etcd (como un static pod):
   ```bash
   sudo crictl ps -a | grep etcd
   sudo crictl logs <etcd-container-id>
   ```
2. Revisar `/etc/kubernetes/manifests/etcd.yaml`.
3. Buscar problemas comunes: rutas incorrectas para `--data-dir`, o certificados que apuntan a rutas inexistentes (ej. `--cert-file=/etc/kubernetes/pki/etcd/server-wrong.crt`).
4. Corregir el archivo estático. El pod se recreará y el clúster se estabilizará.

**Causa Raíz:** Errores de configuración de montajes de volumen o rutas de certificados pasadas como argumentos en el manifiesto estático del etcd.

---

## Ejercicio 14: Logs de un pod (Contenedor múltiple y previous)

**Tiempo estimado:** 2 minutos

**Escenario:**
Un pod llamado `fluentd-sidecar` tiene dos contenedores: `app-web` y `log-agent`. `log-agent` crasheó hace unos minutos y se reinició.

**Solución paso a paso:**
1. Intentar ver logs normales:
   ```bash
   kubectl logs fluentd-sidecar
   ```
   *(Suele pedir que especifiques el contenedor con `-c`)*
2. Ver los logs de la instancia anterior que crasheó para entender por qué:
   ```bash
   kubectl logs fluentd-sidecar -c log-agent --previous
   ```

**Causa Raíz:** Necesario saber extraer los logs de contenedores que ya han terminado o muerto (--previous) en arquitecturas multi-container.

---

## Ejercicio 15: Monitoreo con kubectl top

**Tiempo estimado:** 2 minutos

**Escenario:**
Encuentra el pod en el namespace `kube-system` que está consumiendo más CPU y anota su nombre en `/opt/high_cpu_pod.txt`.

**Solución paso a paso:**
1. Usar `kubectl top` ordenando por CPU:
   ```bash
   kubectl top pods -n kube-system --sort-by=cpu
   ```
2. Observar el primer resultado.
3. Escribir el nombre del pod en el archivo indicado:
   ```bash
   echo "kube-apiserver-master" > /opt/high_cpu_pod.txt
   ```

**Causa Raíz:** Evalúa tu conocimiento usando Metrics Server para troubleshooting de rendimiento (Performance Bottlenecks).

---

## Ejercicio 16: Arreglar un static pod roto

**Tiempo estimado:** 4 minutos

**Escenario:**
Se suponía que un pod estático llamado `nginx-critical` se iba a ejecutar en el `node01`, pero no aparece en `kubectl get pods`.

**Solución paso a paso:**
1. SSH al nodo en cuestión:
   ```bash
   ssh node01
   ```
2. Ir a la carpeta de manifiestos:
   ```bash
   cd /etc/kubernetes/manifests/
   ```
3. Verificar si existe el archivo y editarlo para buscar errores YAML o de formato.
   ```bash
   sudo vi nginx-critical.yaml
   ```
4. Corregir posibles errores (ej. `image: ngin:1.19` en lugar de `nginx`, o indentaciones incorrectas en contenedores).
5. Kubelet procesa cambios inmediatamente. Comprobar:
   ```bash
   crictl ps | grep nginx
   ```

**Causa Raíz:** Los static pods son creados por el kubelet local en base a archivos YAML; si el YAML es inválido, el pod no se crea y no se reporta al API Server.

---

## Ejercicio 17: Pod en estado Terminating atascado

**Tiempo estimado:** 2 minutos

**Escenario:**
El pod `legacy-app` lleva 3 horas en estado `Terminating` y no desaparece.

**Solución paso a paso:**
1. Inspeccionar el pod para entender por qué no se elimina (quizás esperando que un volumen se desmonte o hay un finalizer bloqueado).
2. Forzar la eliminación del pod ignorando el graceful shutdown:
   ```bash
   kubectl delete pod legacy-app --grace-period=0 --force
   ```

**Causa Raíz:** Frecuentemente causado por componentes subyacentes del nodo (CRI, CNI, CSI) que dejan de responder mientras intentan limpiar los recursos, causando que el Pod quede "estancado" esperando la señal de terminación completa.
