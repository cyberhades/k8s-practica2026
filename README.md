# Práctica 2026

El objetivo de esta práctica es desplegar una aplicación web sencilla en Kubernetes **de forma segura**, poniendo en práctica lo visto en clase: espacios de nombres, Pod Security Admission, deployments, probes, contexto de seguridad y servicios.

Para ello necesitarás acceso a un clúster de Kubernetes, puedes usar uno local como `minikube`, o remoto.

La aplicación a desplegar es `tuxotron/demoapp:v1`, una aplicación web que escucha en el puerto `8080` y que, al recibir una petición, devuelve su versión y el nombre del Pod que ha atendido la petición:

```
Version 1
Hostname: demoapp-7d9c8b6f5d-abcde
```

En este mismo repositorio tienes el fichero `kube/deployment.yaml`, que está **incompleto**. Los puntos que debes completar están marcados con comentarios `TODO`.

Se pide lo siguiente:

#### 1. Creación del espacio de nombres

Crear un fichero `kube/namespace.yaml` que defina un espacio de nombres llamado `practica2026` en el que se aplique el perfil de seguridad `restricted` de **Pod Security Admission** en modo forzado (`enforce`). Añade también el modo `warn` con el mismo perfil, para ver los avisos al desplegar.

El espacio de nombres debe crearse **a partir de dicho fichero** (no con `kubectl create ns` ni `kubectl label`):

    kubectl apply -f kube/namespace.yaml

#### 2. Completar el deployment

Modifica el fichero `kube/deployment.yaml` para que:

- Tenga **3 réplicas**.
- **No monte el token** de la cuenta de servicio en los Pods.
- Tenga una **readinessProbe** y una **livenessProbe** de tipo HTTP (`GET /` en el puerto `8080`).
- El contenedor se ejecute con un **contexto de seguridad** que cumpla el perfil `restricted`:
  - No ejecutarse como root (usa el usuario `1000`).
  - No permitir la escalada de privilegios.
  - Eliminar todas las capacidades (`capabilities`).
  - Usar el perfil de seccomp `RuntimeDefault`.
  - Montar el sistema de ficheros raíz como **sólo lectura**.

Aplica el fichero y comprueba que los 3 Pods están en estado `Running` y `READY 1/1`:

    kubectl apply -f kube/deployment.yaml
    kubectl -n practica2026 get po


#### 3. Exponer la aplicación

Crea un fichero `kube/service.yaml` con un servicio de tipo `NodePort` llamado `demoapp`, en el espacio de nombres `practica2026`, que exponga el puerto `8080` de los Pods del deployment.

Comprueba que puedes acceder a la aplicación desde fuera del clúster. Si haces varias peticiones, deberías de ver cómo el valor de `Hostname` cambia, ya que el servicio reparte las peticiones entre las 3 réplicas.

## Entrega

La entrega consistirá en:

- Un fichero `.zip` con los ficheros `kube/namespace.yaml`, `kube/deployment.yaml` (modificado) y `kube/service.yaml`.

## Puntuación de la práctica

- Fichero `namespace.yaml` con Pod Security Admission - 2 puntos
- Réplicas, token de la cuenta de servicio y probes - 3 puntos
- Contexto de seguridad del contenedor - 3 puntos
- Creación del servicio y acceso a la aplicación - 2 puntos

## Notas

Si todo está desplegado de forma correcta, deberías de tener algo parecido a esto:

```
$ kubectl -n practica2026 get deploy,po,svc
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/demoapp   3/3     3            3           2m

NAME                           READY   STATUS    RESTARTS   AGE
pod/demoapp-6b8f7c9d4f-4xkzq   1/1     Running   0          2m
pod/demoapp-6b8f7c9d4f-9pl2m   1/1     Running   0          2m
pod/demoapp-6b8f7c9d4f-tq7vn   1/1     Running   0          2m

NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/demoapp   NodePort   10.104.87.211   <none>        8080:31234/TCP   1m
```

Para acceder a la aplicación con `minikube`:

    curl $(minikube ip):31234

Donde `31234` es el `NodePort` asignado al servicio. Si usas el driver `docker` de minikube (por ejemplo en macOS), usa en su lugar:

    minikube service demoapp -n practica2026 --url

y haz las peticiones a la URL que te devuelva dicho comando.

La imagen `tuxotron/demoapp` está construida para la arquitectura `amd64`. En equipos ARM (por ejemplo, un Mac con Apple Silicon) con minikube y el driver `docker` funciona mediante emulación, aunque los Pods pueden tardar unos segundos más en arrancar.

Documentación útil:

- Pod Security Admission: https://kubernetes.io/docs/concepts/security/pod-security-admission/
- Pod Security Standards: https://kubernetes.io/docs/concepts/security/pod-security-standards/
- Probes: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
- Security Context: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
