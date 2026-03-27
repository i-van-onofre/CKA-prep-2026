Ejercicios prácticos estilo CKA
Gestión de Pods y Deployments

Crear un Pod con una imagen de Nginx.

Escalar un Deployment de 2 a 5 réplicas.

Actualizar la imagen de un Deployment y hacer rollback.

ConfigMaps y Secrets

Crear un ConfigMap con variables de entorno y montarlo en un Pod.

Crear un Secret y usarlo en un contenedor.

Verificar que las variables se cargan correctamente.

Networking y Services

Exponer un Deployment con un Service tipo ClusterIP.

Cambiarlo a NodePort y probar acceso desde fuera del clúster.

Crear un Ingress que dirija tráfico a dos aplicaciones distintas.

Storage

Crear un PersistentVolume (PV) y un PersistentVolumeClaim (PVC).

Montar el PVC en un Pod y verificar que se escribe en el volumen.

Simular un fallo y comprobar que los datos persisten.

RBAC (Roles y permisos)

Crear un ServiceAccount.

Asignarle un Role con permisos para listar Pods.

Probar acceso con kubectl auth can-i.

Troubleshooting

Investigar por qué un Pod no arranca (logs, describe).

Resolver un CrashLoopBackOff.

Diagnosticar problemas de red entre Pods.

Cluster Administration

Agregar un nuevo nodo al clúster.

Drenar un nodo para mantenimiento.

Configurar y verificar el uso de kubectl top con Metrics Server.

Backup y Restore

Hacer backup de recursos con kubectl get all -o yaml > backup.yaml.

Restaurar desde el archivo.

Probar herramientas como etcdctl para respaldar el estado del clúster.