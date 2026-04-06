Deployment
 ├─ apiVersion: apps/v1
 ├─ kind: Deployment
 ├─ metadata
 │    ├─ name
 │    ├─ namespace
 │    └─ labels
 └─ spec
      ├─ replicas: <number of pods>
      ├─ selector
      │     └─ matchLabels
      └─ template
           ├─ metadata
           │     └─ labels
           └─ spec
                 └─ containers
                       ├─ name
                       ├─ image
                       ├─ ports
                       │     └─ containerPort
                       ├─ resources
                       │     ├─ requests (cpu/memory)
                       │     └─ limits (cpu/memory)
                       └─ probes
                             ├─ livenessProbe
                             └─ readinessProbe

---
HPA
 ├─ apiVersion: autoscaling/v2
 ├─ kind: HorizontalPodAutoscaler
 ├─ metadata
 │    └─ name
 └─ spec
      ├─ scaleTargetRef (Deployment to scale)
      │     ├─ apiVersion: apps/v1
      │     ├─ kind: Deployment
      │     └─ name: <deployment-name>
      ├─ minReplicas
      ├─ maxReplicas
      └─ metrics
            └─ type: Resource
                  ├─ name: cpu
                  └─ target: averageUtilization (%)