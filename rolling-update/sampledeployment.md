apiVersion: v1
kind: Service
metadata:
  name: {{ my_service_name}}
  namespace: {{ .namespace }}
  labels:
    ad-app: {{ my_service_name}}
  annotations:
    goodwill.com/ingress.name: {{ my_service_name}}
    goodwill.com/ingress.mgmt: "true"
    goodwill.com/metadata.owner: aakash@goodwill.com
    goodwill.com/metadata.slack: "#dev_support"
    prometheus.io/scrape: "true"
    prometheus.io/path: "/actuator/prometheus"
    prometheus.io/port: "8080"
    getambassador.io/config: |
      apiVersion: ambassador/v0
      kind: Mapping
      name: {{ my_service_name}}
      prefix: /api/v1/tests/config
      rewrite: /api/v1/tests/config
      service: {{ my_service_name}}
spec:
  selector:
    ad-app: {{ my_service_name}}
  ports:
    - name: http-port
      port: 80
      targetPort: http

---

apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: {{ my_service_name}}
  namespace: {{ .namespace }}
  annotations:
    goodwill.com/metadata.owner: aakash@goodwill.com
  labels:
    ad-app: {{ my_service_name}}
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      ad-app: {{ my_service_name}}

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: {{ my_service_name}}-map
  namespace: {{ .namespace }}
  labels:
    ad-app: {{ my_service_name}}
  annotations:
    goodwill.com/metadata.owner: aakash@goodwill.com
    goodwill.com/metadata.github: https://github.com/goodwill/onlinetests
data:
  application.yaml: |-
    environment:
      id: {{ .environment.id }}
    spring:
      data:
        mongodb:
          uri: {{ .mongodb.url }}
          database: ${MONGODB_USER}
      sleuth:
        enabled: {{ .sleuth.enabled }}
        log:
          slf4j:
            enabled: true
        sampler:
          probability: 1
        web:
          skipPattern: /actuator/.*
      zipkin:
        enabled: {{ .zipkin.enabled }}
        baseUrl: {{ .zipkin.baseurl }}
        service:
          name: {{ my_service_name}}
        sender:
          type: web

    goodwill:
      services:
        clients:
          questionPapers:
            interface-name: com.goodwill.questionPapers.questionPapersApi
            base-url: ${questionPapers.baseurl}
    resilience4j:
      circuitbreaker:
        backends:
          questionPapers:
            wait-interval: 60000
            failure-rate-threshold: 50
            ring-buffer-size-in-closed-state: 8
            ring-buffer-size-in-half-open-state: 4
            event-consumer-buffer-size: 100
            register-health-indicator: true
    questionPapers:
      baseurl: {{ .image.library.baseurl }}

    management:
      endpoints:
        metrics:
          enabled: true
        prometheus:
          enabled: true
        web:
          exposure:
           include: "*"
      metrics:
        export:
          prometheus:
            enabled: true
      security:
        enabled: false

    storage:
      type: {{ .storage.type }}
      bucketName: ${STORAGE_BUCKETNAME}
      accessId: ${STORAGE_ACCESSID}
      secretKey: ${STORAGE_SECRETKEY}
      region: ${STORAGE_REGION}
      stsRoleArn: ${STORAGE_STSROLEARN}
      endpointUrl: ${STORAGE_ENDPOINTURL}
      signerType: {{ .storage.signerType }}

    papers:
      retention:
        defaultPeriod: 120

---

apiVersion: persistence.goodwill.com/v1alpha1
kind: MongoDBDatabase
metadata:
  labels:
    ad-app: {{ my_service_name}}
  annotations:
    goodwill.com/metadata.owner: aakash@goodwill.com
  name: {{ my_service_name}}
  namespace: {{ .namespace }}
spec:
  name: question_papers

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ my_service_name}}
  namespace: {{ .namespace }}
  labels:
    ad-app: {{ my_service_name}}
  annotations:
    goodwill.com/metadata.owner: aakash@goodwill.com
    goodwill.com/metadata.github: https://github.com/goodwill/tets
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 3
  progressDeadlineSeconds: 200
  selector:
    matchLabels:
      ad-app: {{ my_service_name}}
  template:
    metadata:
      labels:
        ad-app: {{ my_service_name}}
    spec:
{{ if .cluster.ImagePullSecretRequired }}
      imagePullSecrets:
        - name: {{ .cluster.ImagePullSecretName }}
{{ end }}
      containers:
        - name: {{ my_service_name}}
          image: docker.goodwill.tools/goodwill-itg/questionPapers:{{ .version }}
          securityContext:
            runAsNonRoot: true
            runAsUser: 1000
          resources:
            requests:
              cpu: 100m
              memory: 512Mi
            limits:
              cpu: 2000m
              memory: 512Mi
          volumeMounts:
            - name: tmp
              mountPath: /tmp
          envFrom:
            - secretRef:
                name: {{ my_service_name}}-secrets
          env:
            - name: SPRING_CONFIG_LOCATION
              value: "file:///spring/"
            - name: JAVA_OPTS
              value: -Xms256M -Xmx256M -XX:MaxMetaspaceSize=128M
            - name: MONGODB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ my_service_name}}-mongodb
                  key: password
                  optional: true
            - name: MONGODB_USER
              valueFrom:
                secretKeyRef:
                  name: {{ my_service_name}}-mongodb
                  key: name
                  optional: true
            - name: SPRING_RABBITMQ_USERNAME
              valueFrom:
                secretKeyRef:
                  name: itgconfig-rms-secrets
                  key: rabbit-user
            - name: SPRING_RABBITMQ_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: itgconfig-rms-secrets
                  key: rabbit-password
            - name: SPRING_RABBITMQ_VIRTUAL_HOST
              valueFrom:
                configMapKeyRef:
                  name: itgconfig-rms-config
                  key: vhost
            - name: SPRING_RABBITMQ_HOST
              value: "rabbitmq"
            - name: SPRING_RABBITMQ_PORT
              value: "5672"
          ports:
            - containerPort: 8080
              name: http
          livenessProbe:
            httpGet:
              path: /actuator/health
              port: http
              scheme: HTTP
            initialDelaySeconds: 120
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: http
              scheme: HTTP
            initialDelaySeconds: 120
            timeoutSeconds: 5
          volumeMounts:
            - name: spring-boot-config
              mountPath: /spring
      volumes:
        - name: tmp
          emptyDir: {}
        - name: spring-boot-config
          configMap:
            name: {{ my_service_name}}-map

---

apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: {{ my_service_name}}-secrets
  namespace: {{ .namespace }}
  annotations:
    goodwill.com/metadata.owner: aakash@goodwill.com
spec:
  encryptedData:
    STORAGE_ACCESSID: {{ .storage.accessId }}
    STORAGE_SECRETKEY: {{ .storage.secretKey }}
    STORAGE_REGION: {{ .storage.region }}
    STORAGE_BUCKETNAME: {{ .storage.bucketName }}
    STORAGE_STSROLEARN: {{ .storage.stsRoleArn }}
    STORAGE_ENDPOINTURL: {{ .storage.endpointUrl }}
