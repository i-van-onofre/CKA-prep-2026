# Dominio: Services & Networking (20% del examen CKA)

En este documento se presentan 12 ejercicios prácticos orientados al dominio de Services y Networking para la preparación del examen CKA. 

Documentación oficial útil durante el examen:
- Services: https://kubernetes.io/docs/concepts/services-networking/service/
- Ingress: https://kubernetes.io/docs/concepts/services-networking/ingress/
- Network Policies: https://kubernetes.io/docs/concepts/services-networking/network-policies/
- DNS for Services and Pods: https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

---

## Ejercicio 1: Service ClusterIP
**Tiempo estimado:** 5 minutos

**Enunciado:**
Crea un Deployment llamado `nginx-deploy` en el namespace `default` con la imagen `nginx:alpine` y 2 réplicas. 
Expón el Deployment internamente usando un Service de tipo `ClusterIP` llamado `nginx-svc` en el puerto 80.
Verifica la conectividad al servicio lanzando un pod temporal con la imagen `busybox` y ejecutando un `wget` o `curl` hacia el servicio.

**Solución paso a paso:**
1. Crear el Deployment:
```bash
kubectl create deployment nginx-deploy --image=nginx:alpine --replicas=2
```
2. Crear el Service (ClusterIP es el tipo por defecto):
```bash
kubectl expose deployment nginx-deploy --name=nginx-svc --port=80 --target-port=80
```
*(Alternativa declarativa para el Service)*:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx-deploy
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
3. Verificar el servicio y su IP:
```bash
kubectl get svc nginx-svc
```

**Comando de verificación:**
Lanzar un pod temporal para probar la conectividad:
```bash
kubectl run test-pod --image=busybox:1.28 --rm -it --restart=Never -- wget -qO- http://nginx-svc
```

---

## Ejercicio 2: Service NodePort
**Tiempo estimado:** 5 minutos

**Enunciado:**
Crea un Deployment llamado `webapp-deploy` usando la imagen `httpd:2.4`.
Crea un Service llamado `webapp-nodeport` para exponer el Deployment anterior en el puerto `80`.
Asegúrate de que el Service esté accesible desde el exterior en el puerto `30080` de los nodos del clúster.

**Solución paso a paso:**
1. Crear el Deployment:
```bash
kubectl create deployment webapp-deploy --image=httpd:2.4
```
2. Crear un archivo yaml para el servicio, ya que especificar el nodePort directamente desde el comando imperativo completo a veces requiere modificar el YAML:
```bash
kubectl create service nodeport webapp-nodeport --tcp=80:80 --dry-run=client -o yaml > nodeport.yaml
```
3. Editar `nodeport.yaml` para añadir el `nodePort: 30080`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-nodeport
spec:
  type: NodePort
  selector:
    app: webapp-deploy
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```
4. Aplicar el YAML:
```bash
kubectl apply -f nodeport.yaml
```

**Comando de verificación:**
```bash
kubectl get svc webapp-nodeport
# Deberías ver que expone el puerto 80:30080/TCP
```

---

## Ejercicio 3: Crear Ingress con reglas Path-based
**Tiempo estimado:** 8 minutos

**Enunciado:**
Existen dos servicios en el namespace `ingress-space`: `video-service` en el puerto 8080 y `audio-service` en el puerto 9090.
Crea un recurso Ingress llamado `media-ingress` en el namespace `ingress-space` que enrute el tráfico de la siguiente manera:
- Path `/video` -> `video-service:8080`
- Path `/audio` -> `audio-service:9090`
Asegúrate de especificar `nginx` como IngressClass si es necesario, y usa `Prefix` como `pathType`.

**Solución paso a paso:**
1. Crear el namespace (si no existe para probar):
```bash
kubectl create ns ingress-space
```
2. Crear el manifiesto YAML para el Ingress:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: media-ingress
  namespace: ingress-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /video
        pathType: Prefix
        backend:
          service:
            name: video-service
            port:
              number: 8080
      - path: /audio
        pathType: Prefix
        backend:
          service:
            name: audio-service
            port:
              number: 9090
```
3. Aplicar el Ingress:
```bash
kubectl apply -f ingress.yaml
```

**Comando de verificación:**
```bash
kubectl describe ingress media-ingress -n ingress-space
```

---

## Ejercicio 4: Ingress con TLS
**Tiempo estimado:** 8 minutos

**Enunciado:**
Crea un secreto de tipo TLS llamado `secure-cert` usando los archivos `tls.crt` y `tls.key` (para este ejercicio asume que ya los generaste o usa comandos openssl rápidos).
Crea un Ingress llamado `secure-ingress` que exponga un servicio existente llamado `secure-service` (puerto 80) a través del host `secure.example.com` usando el certificado TLS creado.

**Solución paso a paso:**
1. (Opcional) Generar certificados autofirmados para el ejercicio:
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=secure.example.com"
```
2. Crear el Secret TLS:
```bash
kubectl create secret tls secure-cert --key tls.key --cert tls.crt
```
3. Crear el YAML del Ingress `secure-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
spec:
  tls:
  - hosts:
      - secure.example.com
    secretName: secure-cert
  rules:
  - host: secure.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: secure-service
            port:
              number: 80
```
4. Aplicar:
```bash
kubectl apply -f secure-ingress.yaml
```

**Comando de verificación:**
```bash
kubectl get ingress secure-ingress
kubectl describe ingress secure-ingress | grep TLS
```

---

## Ejercicio 5: DNS de Kubernetes (FQDN)
**Tiempo estimado:** 5 minutos

**Enunciado:**
Tienes un servicio llamado `db-service` en el namespace `backend-ns`.
Despliega un Pod llamado `dns-test` con la imagen `busybox:1.28` en el namespace `default`.
Desde el pod `dns-test`, realiza un `nslookup` usando el Fully Qualified Domain Name (FQDN) del `db-service` para verificar la resolución DNS.
Guarda la salida del nslookup en el archivo `/tmp/dns-resolve.txt` dentro de tu nodo maestro o local.

**Solución paso a paso:**
1. Crear el namespace y un servicio dummy:
```bash
kubectl create ns backend-ns
kubectl create deployment db --image=redis -n backend-ns
kubectl expose deployment db --name=db-service --port=6379 -n backend-ns
```
2. Ejecutar el pod temporal para hacer nslookup y guardar la salida:
```bash
kubectl run dns-test --image=busybox:1.28 -n default --rm -it --restart=Never -- nslookup db-service.backend-ns.svc.cluster.local > /tmp/dns-resolve.txt
```

**Comando de verificación:**
```bash
cat /tmp/dns-resolve.txt
```

---

## Ejercicio 6: NetworkPolicy Ingress
**Tiempo estimado:** 8 minutos

**Enunciado:**
En el namespace `api-ns`, crea una NetworkPolicy llamada `api-allow`.
Esta política debe permitir tráfico entrante (Ingress) al puerto `8080` de los pods con el label `app=api-server`, **solo** desde aquellos pods que tengan el label `role=frontend`.

**Solución paso a paso:**
1. Crear el YAML `api-allow.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
  namespace: api-ns
spec:
  podSelector:
    matchLabels:
      app: api-server
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 8080
```
2. Aplicar la política:
```bash
kubectl apply -f api-allow.yaml
```

**Comando de verificación:**
```bash
kubectl describe networkpolicy api-allow -n api-ns
```

---

## Ejercicio 7: NetworkPolicy Egress
**Tiempo estimado:** 10 minutos

**Enunciado:**
Crea una NetworkPolicy llamada `restrict-egress` en el namespace `secure-ns` que aplique a pods con `env=prod`.
Estos pods solo deben tener permitido tráfico saliente (Egress) hacia el puerto `53` (TCP/UDP para DNS) y hacia el CIDR `10.0.0.0/24` en el puerto `443`.
Cualquier otro tráfico de salida debe estar bloqueado.

**Solución paso a paso:**
1. Crear el archivo `restrict-egress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress
  namespace: secure-ns
spec:
  podSelector:
    matchLabels:
      env: prod
  policyTypes:
    - Egress
  egress:
    - ports:
        - protocol: TCP
          port: 53
        - protocol: UDP
          port: 53
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 443
```
2. Aplicar la NetworkPolicy:
```bash
kubectl apply -f restrict-egress.yaml
```

**Comando de verificación:**
```bash
kubectl describe networkpolicy restrict-egress -n secure-ns
```

---

## Ejercicio 8: NetworkPolicy Deny All
**Tiempo estimado:** 5 minutos

**Enunciado:**
Asegura el namespace `project-x` creando una NetworkPolicy llamada `default-deny` que bloquee por defecto todo el tráfico de entrada (Ingress) y salida (Egress) para todos los pods en este namespace.

**Solución paso a paso:**
1. Crear el YAML `default-deny.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: project-x
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```
2. Aplicar:
```bash
kubectl apply -f default-deny.yaml
```

**Comando de verificación:**
```bash
kubectl get networkpolicy default-deny -n project-x -o yaml
```

---

## Ejercicio 9: Service sin selector y Endpoints
**Tiempo estimado:** 8 minutos

**Enunciado:**
Crea un Service llamado `external-db-svc` en el namespace `default` pero **sin un selector**.
A continuación, crea manualmente el recurso `Endpoints` correspondiente que enrute el tráfico de ese servicio hacia la IP `192.168.1.100` en el puerto `3306`.

**Solución paso a paso:**
1. Crear el archivo `svc-endpoints.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db-svc
spec:
  ports:
    - port: 3306
      targetPort: 3306
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-db-svc
subsets:
  - addresses:
      - ip: 192.168.1.100
    ports:
      - port: 3306
```
2. Aplicar ambos:
```bash
kubectl apply -f svc-endpoints.yaml
```

**Comando de verificación:**
```bash
kubectl describe svc external-db-svc
kubectl get endpoints external-db-svc
```

---

## Ejercicio 10: Depurar conectividad de red
**Tiempo estimado:** 6 minutos

**Enunciado:**
Un desarrollador se queja de que su frontend no puede comunicarse con su backend. 
Despliega un pod interactivo (imagen `busybox` o `alpine`) llamado `net-debug` en el namespace `default`.
Usa este pod para probar la resolución DNS con `nslookup kubernetes.default` y probar conectividad con `wget --spider kubernetes.default.svc.cluster.local`.

**Solución paso a paso:**
1. Crear el pod iteractivo:
```bash
kubectl run net-debug --image=busybox:1.28 --rm -it --restart=Never -- sh
```
2. Una vez dentro del shell interactivo, ejecutar los comandos:
```sh
nslookup kubernetes.default
wget --spider --no-check-certificate https://kubernetes.default.svc.cluster.local
exit
```

**Comando de verificación:**
El shell saldrá tras el `exit`. Si nslookup funciona, el DNS del cluster está saludable.

---

## Ejercicio 11: CoreDNS Troubleshooting
**Tiempo estimado:** 8 minutos

**Enunciado:**
Si notas que la resolución DNS dentro de tu cluster (por ejemplo, desde el pod `net-debug` del ejercicio anterior) falla, describe qué componentes revisarías.
Identifica los pods de CoreDNS en tu clúster, extrae sus logs e identifica la ConfigMap de CoreDNS.

**Solución paso a paso:**
1. Comprobar los pods de CoreDNS (generalmente en `kube-system`):
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```
2. Ver logs de CoreDNS en busca de errores:
```bash
kubectl logs -n kube-system -l k8s-app=kube-dns
```
3. Verificar si el servicio de DNS existe y tiene endpoints:
```bash
kubectl get svc -n kube-system kube-dns
kubectl get endpoints -n kube-system kube-dns
```
4. Inspeccionar la configuración de CoreDNS (Corefile):
```bash
kubectl get configmap coredns -n kube-system -o yaml
```

**Comando de verificación:**
Un simple nslookup en cualquier pod (ej. `kubectl run test --image=busybox --rm -it -- nslookup google.com`) validará si está operativo de nuevo.

---

## Ejercicio 12: Exponer servicio con ExternalName
**Tiempo estimado:** 5 minutos

**Enunciado:**
El equipo necesita que los pods del cluster accedan a una base de datos externa cuyo dominio es `db.external.company.com`, pero usando un nombre interno de kubernetes llamado `legacy-db` dentro del namespace `default`.
Crea un Service llamado `legacy-db` de tipo `ExternalName` que cumpla con esta función.

**Solución paso a paso:**
1. Crear el Service usando un YAML `external-name.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: legacy-db
  namespace: default
spec:
  type: ExternalName
  externalName: db.external.company.com
```
2. Aplicar el YAML:
```bash
kubectl apply -f external-name.yaml
```
*(Alternativa imperativa, aunque menos común para ExternalName)*:
```bash
kubectl create service externalname legacy-db --tcp=80:80 --external-name=db.external.company.com
# Nota: La creación imperativa pura para ExternalName puede variar, el YAML es la mejor y más segura práctica.
```

**Comando de verificación:**
```bash
kubectl get svc legacy-db
# Comprobar resolución DNS (mostrará el CNAME):
kubectl run dns-test --image=busybox:1.28 --rm -it -- nslookup legacy-db
```
