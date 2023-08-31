## Задание 1. Подготовить Helm-чарт для приложения
--
1. Скачал Helm по оф. инструкции https://helm.sh/docs/intro/install/, создали чарт и шаблонизировали приложение
````
vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ ll
total 28
drwxr-xr-x 4 vagrant vagrant 4096 Aug 31 14:22 ./
drwxrwxr-x 3 vagrant vagrant 4096 Aug 30 13:03 ../
drwxr-xr-x 2 vagrant vagrant 4096 Aug 30 13:03 charts/
-rw-r--r-- 1 vagrant vagrant 1154 Aug 31 14:22 Chart.yaml
-rw-r--r-- 1 vagrant vagrant  492 Aug 31 11:30 .helmignore
drwxr-xr-x 3 vagrant vagrant 4096 Aug 31 12:33 templates/
-rw-r--r-- 1 vagrant vagrant 2243 Aug 31 12:40 values.yaml


vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ ll templates/
total 44
drwxr-xr-x 3 vagrant vagrant 4096 Aug 31 12:33 ./
drwxr-xr-x 4 vagrant vagrant 4096 Aug 31 14:22 ../
-rw-r--r-- 1 vagrant vagrant 1915 Aug 30 13:03 deployment.yaml
-rw-rw-r-- 1 vagrant vagrant  662 Aug 31 12:19 deploy.yaml
-rw-r--r-- 1 vagrant vagrant 1892 Aug 30 13:03 _helpers.tpl
-rw-r--r-- 1 vagrant vagrant 1024 Aug 30 13:03 hpa.yaml
-rw-r--r-- 1 vagrant vagrant 2101 Aug 30 13:03 ingress.yaml
-rw-r--r-- 1 vagrant vagrant 1791 Aug 30 13:03 NOTES.txt
-rw-r--r-- 1 vagrant vagrant  342 Aug 30 13:03 serviceaccount.yaml
-rw-r--r-- 1 vagrant vagrant  419 Aug 31 12:16 service.yaml
drwxr-xr-x 2 vagrant vagrant 4096 Aug 30 13:03 tests/

Для простоты работы создал только 2 объектка Deployment и Service, остальные файлы добавил в файл .helmignore

vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ nano .helmignore
...
# Standart file in template directory
templates/deployment.yaml
templates/hpa.yaml
templates/ingress.yaml
templates/serviceaccount.yaml
tests/

vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ cat templates/deploy.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  {{- with .Values.metadata }}
    {{- toYaml . | nindent 2 }}
  {{- end }}
  labels:
    app: {{ .Values.metadata.labels.app }}
    {{- include "netology-test-repo.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.metadata.labels.app }}
  template:
    metadata:
      labels:
        app: {{ .Values.metadata.labels.app }}
    {{- with .Values.spec }}
    spec:
      {{- toYaml . | nindent 6 }}
    {{- end }}
      {{- with .Values.volumes }}
      volumes:
        {{- toYaml . | nindent 6 }}
      {{- end }}
      restartPolicy: Always

vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ cat templates/service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.metadata.name }}-svc
  labels:
    {{- toYaml .Values.metadata.labels | nindent 4 }}
    {{- include "netology-test-repo.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- toYaml .Values.metadata.labels | nindent 4 }}

vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ cat values.yaml
---
# Default values for netology-test-repo.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 2

metadata:
  name: netology-test-repo-nginx
  labels:
    app: nginx

spec:
  containers:
  - name: nginx
    image: nginx:1.24.0
    ports:
    - containerPort: 8080
      name: http-port
    volumeMounts:
    - name: vol
      mountPath: /output
    env:
    - name: http-port
      value: "8080"

volumes:
- name: vol
  hostPath:
    path: /var/data

image:
  repository: nginx
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 8080


ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}
````

## Задание 2. Запустить две версии в разных неймспейсах
--
1. Проверяем созданные файл Helm'мом и пробуем задеплоит в разные неймспейсы
````
vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ helm template netology-test . -f ./values.yaml
---
# Source: netology-test-repo/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: netology-test-repo-nginx-svc
  labels:
    app: nginx
    helm.sh/chart: netology-test-repo-0.1.0
    app.kubernetes.io/name: netology-test-repo
    app.kubernetes.io/instance: netology-test
    app.kubernetes.io/version: "1.24.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: nginx
---
# Source: netology-test-repo/templates/deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: netology-test-repo-nginx
  labels:
    app: nginx
    helm.sh/chart: netology-test-repo-0.1.0
    app.kubernetes.io/name: netology-test-repo
    app.kubernetes.io/instance: netology-test
    app.kubernetes.io/version: "1.24.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - env:
        - name: http-port
          value: "8080"
        image: nginx:1.24.0
        name: nginx
        ports:
        - containerPort: 8080
          name: http-port
        volumeMounts:
        - mountPath: /output
          name: vol
      volumes:
      - hostPath:
          path: /var/data
        name: vol
      restartPolicy: Always

vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ helm upgrade --install netology-test . --namespace app1 --create-namespace -f values.yaml
Release "netology-test" does not exist. Installing it now.
NAME: netology-test
LAST DEPLOYED: Thu Aug 31 14:31:35 2023
NAMESPACE: app1
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace app1 -l "app.kubernetes.io/name=netology-test-repo,app.kubernetes.io/instance=netology-test" -o jsonpath="{.items[0].metadata.name}")

  export CONTAINER_PORT=$(kubectl get pod --namespace app1 $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace app1 port-forward $POD_NAME 8080:$CONTAINER_PORT

vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ helm list -n app1
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
netology-test   app1            1               2023-08-31 14:31:35.893918165 +0000 UTC deployed        netology-test-repo-0.1.0        1.24.0

agrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ helm upgrade --install netology-test . --namespace app1 --create-namespace -f values.yaml
Release "netology-test" has been upgraded. Happy Helming!
NAME: netology-test
LAST DEPLOYED: Thu Aug 31 14:32:35 2023
NAMESPACE: app1
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace app1 -l "app.kubernetes.io/name=netology-test-repo,app.kubernetes.io/instance=netology-test" -o jsonpath="{.items[0].metadata.name}")

  export CONTAINER_PORT=$(kubectl get pod --namespace app1 $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace app1 port-forward $POD_NAME 8080:$CONTAINER_PORT

vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ helm list -n app1
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
netology-test   app1            2               2023-08-31 14:32:35.010098342 +0000 UTC deployed        netology-test-repo-0.1.0        1.24.0

vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ helm upgrade --install netology-test . --namespace app2 --create-namespace -f values.yaml
Release "netology-test" does not exist. Installing it now.
NAME: netology-test
LAST DEPLOYED: Thu Aug 31 14:33:08 2023
NAMESPACE: app2
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace app2 -l "app.kubernetes.io/name=netology-test-repo,app.kubernetes.io/instance=netology-test" -o jsonpath="{.items[0].metadata.name}")

  export CONTAINER_PORT=$(kubectl get pod --namespace app2 $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace app2 port-forward $POD_NAME 8080:$CONTAINER_PORT

vagrant@vagrant:~/kubernetes/kuber_Helm/netology-test-repo$ helm list -n app2
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
netology-test   app2            1               2023-08-31 14:33:08.417106211 +0000 UTC deployed        netology-test-repo-0.1.0        1.24.0
````
