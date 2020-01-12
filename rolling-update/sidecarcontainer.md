# Deployment using two pods aka sidecar container deployments

```
apiVersion: v1
kind: Service
metadata:
  name: pdf-generator
  labels:
    ad-app: pdf-generator
  annotations:
    goodwill.com/ingress.name: pdf-generator
    goodwill.com/ingress.mgmt: "true"
    goodwill.com/metadata.owner: goodwill_pune@goodwill.com
    goodwill.com/metadata.slack: "#dev-alerts"
    prometheus.io/scrape: "true"
    prometheus.io/path: "/prometheus"
    prometheus.io/port: "8080"
spec:
  selector:
    ad-app: pdf-generator
  ports:
    - name: http-port
      port: 80
      targetPort: http

---

apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: pdf-generator
  annotations:
    goodwill.com/metadata.owner: goodwill_pune@goodwill.com
  labels:
    ad-app: pdf-generator
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      ad-app: pdf-generator

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pdf-generator
  labels:
    ad-app: pdf-generator
  annotations:
    goodwill.com/metadata.owner: goodwill_pune@goodwill.com
    goodwill.com/metadata.github: https://github.com/goodwill/pdf-generator
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
      ad-app: pdf-generator
  template:
    metadata:
      labels:
        ad-app: pdf-generator
    spec:
      containers:
        - name: pdf-generator
          image: docker.goodwill.tools/goodwill/pdf-generator:{{ .version }}
          securityContext:
            runAsNonRoot: true
            runAsUser: 1000
          resources:
            requests:
              cpu: 250m
              memory: 512Mi
            limits:
              cpu: 2000m
              memory: 512Mi
          volumeMounts:
              - name: tmp
                mountPath: /tmp
          env:
            - name: NODE_ENV
              value: "production"
            - name: CHROME_WS_URL
              value: 'ws://localhost:3000'
            - name: BODY_PARSER_LIMIT
              value: '50mb'
            - name: ZIPKIN_BASEURL
              value: {{ .zipkin.baseurl }}
            - name: LOG_LEVEL
              value: {{ .logging.pdfgen.level }}
          ports:
            - containerPort: 8080
              name: http
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 60
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 60
            timeoutSeconds: 5

        - name: chrome-headless
          image: browserless/chrome:release-puppeteer-1.9.0
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
          ports:
            - containerPort: 3000
              name: chrome-port
          env:
            - name: DEBUG
              value: {{ .logging.chrome.debug }}
            - name: PREBOOT_CHROME
              value: 'true'
            - name: MAX_CONCURRENT_SESSIONS
              value: '5'
            - name: MAX_QUEUE_LENGTH
              value: '5'
          livenessProbe:
            httpGet:
              path: /config
              port: 3000
              scheme: HTTP
            initialDelaySeconds: 120
            timeoutSeconds: 5
      volumes:
        - name: tmp
          emptyDir: {}
```
