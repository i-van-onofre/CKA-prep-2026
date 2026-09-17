# Dominio: Storage (10% del examen CKA)

Este documento contiene 8 ejercicios prácticos enfocados en el dominio de Storage (Almacenamiento) para la preparación del examen CKA. En el examen real, las tareas de almacenamiento requieren entender cómo conectar Pods con volúmenes persistentes y efímeros.

---

## Ejercicio 1: PV y PVC estático
**Tiempo estimado:** 8 minutos

**Enunciado:**
Crea un PersistentVolume llamado `pv-static` con las siguientes características:
- Tipo: `hostPath` apuntando al directorio `/opt/vol-data` del nodo.
- Capacidad: `1Gi`
- Modo de acceso: `ReadWriteOnce`

Luego, crea un PersistentVolumeClaim llamado `pvc-static` que solicite `1Gi` y se enlace al PV anterior. Finalmente, crea un Pod llamado `pod-storage` (imagen `nginx`) que monte el PVC en la ruta `/usr/share/nginx/html`.

**Documentación K8s:** [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

**Solución paso a paso:**

1. Crear el manifiesto para el PV y el PVC (`storage.yaml`):
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-static
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /opt/vol-data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-static
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

2. Aplicar el manifiesto:
```bash
kubectl apply -f storage.yaml
```

3. Crear el pod usando un comando imperativo para generar el yaml base y luego editarlo (`pod.yaml`):
```bash
kubectl run pod-storage --image=nginx --dry-run=client -o yaml > pod.yaml
```

4. Editar `pod.yaml` para incluir el volumen:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-storage
  name: pod-storage
spec:
  containers:
  - image: nginx
    name: pod-storage
    volumeMounts:
    - name: data-vol
      mountPath: /usr/share/nginx/html
  volumes:
  - name: data-vol
    persistentVolumeClaim:
      claimName: pvc-static
```

5. Aplicar el Pod:
```bash
kubectl apply -f pod.yaml
```

**Comandos de verificación:**
```bash
kubectl get pv pv-static
kubectl get pvc pvc-static
kubectl describe pod pod-storage | grep -A 2 Mounts
```

---

## Ejercicio 2: StorageClass y provisión dinámica
**Tiempo estimado:** 5 minutos

**Enunciado:**
Crea una StorageClass llamada `fast-sc` que use el provisionador `kubernetes.io/no-provisioner` (o el provisionador por defecto de tu clúster como `k8s.io/minikube-hostpath`) con `volumeBindingMode: WaitForFirstConsumer`.
Crea un PVC llamado `pvc-dynamic` de `2Gi` que use esta StorageClass.
*Nota: En un clúster real con un provisionador válido, el PV se creará automáticamente.*

**Documentación K8s:** [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)

**Solución paso a paso:**

1. Crear manifiesto de StorageClass (`sc.yaml`):
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

2. Crear manifiesto del PVC (`pvc-dynamic.yaml`):
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-sc
  resources:
    requests:
      storage: 2Gi
```

3. Aplicar los manifiestos:
```bash
kubectl apply -f sc.yaml
kubectl apply -f pvc-dynamic.yaml
```

**Comandos de verificación:**
```bash
kubectl get sc fast-sc
kubectl get pvc pvc-dynamic
# Status estará en "Pending" esperando que un Pod consuma el PVC debido a WaitForFirstConsumer
```

---

## Ejercicio 3: Expandir PVC
**Tiempo estimado:** 5 minutos

**Enunciado:**
Asumiendo que tienes una StorageClass llamada `expand-sc` configurada con `allowVolumeExpansion: true`, y un PVC llamado `pvc-expand` creado a partir de ella con un tamaño de `1Gi`. 
Incrementa el tamaño del PVC `pvc-expand` a `2Gi`.

**Documentación K8s:** [Expanding Persistent Volumes Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)

**Solución paso a paso:**

1. Puedes editar el PVC directamente con:
```bash
kubectl edit pvc pvc-expand
```

2. Modifica la sección de `resources.requests.storage`:
```yaml
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi # Cambiar de 1Gi a 2Gi
  storageClassName: expand-sc
```
Guarda y sal del editor.

**Comandos de verificación:**
```bash
kubectl get pvc pvc-expand
kubectl describe pvc pvc-expand | grep Capacity
```
*(Nota: Puede que requiera que el Pod atado al PVC se reinicie para reflejar el cambio en el file system dependiendo del plugin de volumen).*

---

## Ejercicio 4: Volumen emptyDir
**Tiempo estimado:** 7 minutos

**Enunciado:**
Crea un Pod llamado `shared-pod` con dos contenedores:
1. `writer`: imagen `busybox`, ejecuta el comando `/bin/sh -c "while true; do date >> /shared/date.txt; sleep 5; done"`
2. `reader`: imagen `nginx`
Ambos contenedores deben compartir un volumen de tipo `emptyDir`.
- El `writer` lo monta en `/shared`.
- El `reader` lo monta en `/usr/share/nginx/html`.

**Documentación K8s:** [Volumes - emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

**Solución paso a paso:**

1. Crear el manifiesto (`emptydir-pod.yaml`):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ["/bin/sh", "-c", "while true; do date >> /shared/date.txt; sleep 5; done"]
    volumeMounts:
    - name: shared-vol
      mountPath: /shared
  - name: reader
    image: nginx
    volumeMounts:
    - name: shared-vol
      mountPath: /usr/share/nginx/html
  volumes:
  - name: shared-vol
    emptyDir: {}
```

2. Aplicar el manifiesto:
```bash
kubectl apply -f emptydir-pod.yaml
```

**Comandos de verificación:**
```bash
kubectl exec shared-pod -c reader -- cat /usr/share/nginx/html/date.txt
```

---

## Ejercicio 5: Volumen hostPath
**Tiempo estimado:** 5 minutos

**Enunciado:**
Crea un Pod llamado `node-logger` utilizando la imagen `fluentd`. 
Este pod debe montar el directorio del nodo `/var/log` en la ruta interna del contenedor `/logs` utilizando un volumen de tipo `hostPath`.

**Documentación K8s:** [Volumes - hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)

**Solución paso a paso:**

1. Generar la plantilla base:
```bash
kubectl run node-logger --image=fluentd --dry-run=client -o yaml > logger-pod.yaml
```

2. Editar `logger-pod.yaml` para añadir los volúmenes:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: node-logger
  name: node-logger
spec:
  containers:
  - image: fluentd
    name: node-logger
    volumeMounts:
    - name: node-logs
      mountPath: /logs
  volumes:
  - name: node-logs
    hostPath:
      path: /var/log
      type: Directory
```

3. Aplicar y crear el pod:
```bash
kubectl apply -f logger-pod.yaml
```

**Comandos de verificación:**
```bash
kubectl describe pod node-logger
kubectl exec node-logger -- ls /logs
```

---

## Ejercicio 6: PV con diferentes ReclaimPolicy
**Tiempo estimado:** 8 minutos

**Enunciado:**
Crea dos PersistentVolumes:
1. `pv-retain`: Capacidad `1Gi`, `hostPath: /data/retain`, `persistentVolumeReclaimPolicy: Retain`.
2. `pv-delete`: Capacidad `1Gi`, `hostPath: /data/delete`, `persistentVolumeReclaimPolicy: Delete`.
*Entiende la diferencia: Si un PVC se ata y luego se borra, en `Retain` el PV queda como "Released" y los datos se preservan pero no se puede volver a usar automáticamente. En `Delete`, el PV y los datos subyacentes se borran (dependiendo del provisionador, aunque hostPath por defecto no los borra físicamente).*

**Documentación K8s:** [Reclaiming](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming)

**Solución paso a paso:**

1. Crear el manifiesto (`reclaim-pvs.yaml`):
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-retain
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-delete
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  hostPath:
    path: /data/delete
```

2. Aplicar el archivo:
```bash
kubectl apply -f reclaim-pvs.yaml
```

**Comandos de verificación:**
```bash
kubectl get pv pv-retain pv-delete -o custom-columns=NAME:.metadata.name,RECLAIMPOLICY:.spec.persistentVolumeReclaimPolicy
```

---

## Ejercicio 7: Troubleshooting PVC Pending
**Tiempo estimado:** 10 minutos

**Enunciado:**
Un desarrollador ha creado un PVC llamado `pvc-app-data` pidiendo `5Gi` con AccessMode `ReadWriteMany`, pero se queda en estado `Pending`.
Hay un PV disponible llamado `pv-app-data` de `5Gi`. 
Investiga por qué el PVC está en Pending, corrige el problema (puedes recrear el PVC o el PV según sea necesario) para que el PVC quede en estado `Bound`.

**Documentación K8s:** [Troubleshooting Storage](https://kubernetes.io/docs/tasks/administer-cluster/troubleshooting/)

**Solución paso a paso (Simulación):**

1. Inspeccionar el PVC:
```bash
kubectl describe pvc pvc-app-data
# El mensaje dirá algo como: "Failed to bind volumes: ... no matches for volume access modes" 
```

2. Inspeccionar los PVs disponibles:
```bash
kubectl get pv pv-app-data -o yaml | grep -A 2 accessModes
# Descubrirás que el PV probablemente tenga "ReadWriteOnce", 
# mientras que el PVC pidió "ReadWriteMany".
```

3. Recrear el PVC (o el PV) para que los modos de acceso y capacidades coincidan (los PVC no pueden editar su accessMode si ya se enviaron, así que hay que borrarlo y volverlo a crear):
```bash
kubectl delete pvc pvc-app-data
```

4. Crear el PVC correcto (`pvc-fixed.yaml`):
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-app-data
spec:
  accessModes:
    - ReadWriteOnce # Ajustado para que coincida con el PV
  resources:
    requests:
      storage: 5Gi
```

5. Aplicar la solución:
```bash
kubectl apply -f pvc-fixed.yaml
```

**Comandos de verificación:**
```bash
kubectl get pvc pvc-app-data
# Debe mostrar STATUS: Bound
```

---

## Ejercicio 8: VolumeMount subPath
**Tiempo estimado:** 7 minutos

**Enunciado:**
Crea un ConfigMap llamado `app-config` con el contenido `config.json: '{"env":"prod"}'`.
Luego, crea un Pod llamado `subpath-pod` usando la imagen `nginx`.
Monta **únicamente** el archivo `config.json` dentro del contenedor en la ruta `/etc/nginx/config.json`, asegurándote de usar `subPath` para no sobrescribir o limpiar ningún otro archivo en el directorio `/etc/nginx/`.

**Documentación K8s:** [Using subPath](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath)

**Solución paso a paso:**

1. Crear el ConfigMap de forma imperativa:
```bash
kubectl create configmap app-config --from-literal=config.json='{"env":"prod"}'
```

2. Crear el manifiesto del Pod (`pod-subpath.yaml`):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: subpath-pod
spec:
  containers:
  - name: subpath-container
    image: nginx
    volumeMounts:
    - name: config-vol
      mountPath: /etc/nginx/config.json
      subPath: config.json
  volumes:
  - name: config-vol
    configMap:
      name: app-config
```

3. Aplicar el Pod:
```bash
kubectl apply -f pod-subpath.yaml
```

**Comandos de verificación:**
```bash
# Verificar que el archivo existe y contiene lo esperado
kubectl exec subpath-pod -- cat /etc/nginx/config.json

# Verificar que los demás archivos de nginx siguen en /etc/nginx/ (por ejemplo, nginx.conf)
kubectl exec subpath-pod -- ls /etc/nginx/ | grep nginx.conf
```
