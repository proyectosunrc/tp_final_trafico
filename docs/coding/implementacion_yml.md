

## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: miapp-app
spec:
  replicas: 2 # Número de réplicas del pod
  selector:
    matchLabels:
      app: miapp-app
  template:
    metadata:
      labels:
        app: miapp-app
    spec:
      containers:
        - name: miapp-app
          image: ramirotizzian1/miapp:latest
          ports:
            - containerPort: 8000  # Exponemos el puerto 8000 de la aplicación
          resources:
            requests:
              cpu: 500m #CPU asignada a cada pod, puede ser menor al límite máximo
            limits:
              cpu: 1000m #CPU máxima permitida a cada pod
```
*Explicacion basica de funcionamiento*

## Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: miapp-service
  annotations:
    service.kubernetes.io/ipvs-scheduler: "lc"
spec:
  selector:
    app: miapp-app
  ports:
    - protocol: TCP
      port: 80  # El puerto que se expondrá internamente
      targetPort: 8000  # El puerto de la aplicación FastAPI
  type: ClusterIP
```

## Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: miapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/load-balance: "round_robin"
spec:
  rules:
    - host: miapp.local  # Puedes usar cualquier nombre de dominio, en este caso es miapp.local
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: miapp-service
              port:
                number: 80
```


## Horizontal POD Autoescaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: miapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: miapp-app
  minReplicas: 1
  maxReplicas: 6
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 25  # Escala si el uso promedio de CPU excede el 25%


```