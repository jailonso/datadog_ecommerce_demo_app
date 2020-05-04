# Datadog Ecommerce Demo App

We are using ecommerce dogstore demo app and Datadog to show different use cases. 

In this case we will use Datadog cluster agent, which altough it requires more step than the node agent it has several benefits:
- Reduce API server Load
- Use Datadog to Autoscale Pods
- Run service checks against k8s endpoints.

More info here. https://docs.datadoghq.com/agent/cluster_agent/

We will use k8s v1.15.11-gke.11


# Instructions
0. Deploy kube-state-metrics y metric server

```
kubectl apply -f k8s_manifests/kube-state-metric/
kubectl apply -f k8s_manifests/metric-server.yaml
```
1. Deploy RBAC:
```
kubectl apply -f k8s_manifests/datadog/serviceaccount.yaml
```
2. Secure the connection between cluster agent and node agent
```
kubectl create secret generic datadog-auth-token --from-literal token="<ThirtyX2XcharactersXlongXtoken>"
```
3. Create secret for Datadog API and APP Key (Open Datadog application and navigate to Integrations -> APIs. Click on Applications Keys and generate a new application key and a new API Key)
```
kubectl create secret generic datadog-app-key --from-literal app-key=<YOUR_NEWLY_GENERATED_DATADOG_APP_KEY>
kubectl create secret generic datadog-secret --from-literal api-key=<YOUR_DATADOG_API_KEY>
```
4. Deploy Cluster Agent

Modify tags as needed:
```
          - name: DD_CLUSTER_NAME
            value: kubernetes
          - name: DD_ENV
            value: "ruby-shop"
          - name: DD_TAGS
            value: "env:ruby-shop"
  ```
  ```
 kubectl apply -f k8s_manifests/datadog/datadog-cluster-agent.yaml
 ```
5. Deploy Node agent:
```
kubectl apply -f  k8s_manifests/datadog/datadog-agent-with-cluster-agent.yaml
```
6. Check status:
```
kubectl exec -ti $(kubectl get pods -l app=datadog-agent -o jsonpath='{.items[0].metadata.name}') -- agent status
```

7. Deploy application
```
kubectl apply -f k8s_manifests/app/
```

8. We will also need to give the cluster agent some permissions to act as an External Metrics server. Give those permissions executing the following command:
 ```
kubectl apply -f k8s_manifests/datadog/cluster-agent-external-metrics.yaml
```
9. Create SLOs with errors and latency below 7 and check latency  slos is failing.

10. Increase traffic and check if SLO latency is failing
```
kubectl apply -f k8s_manifests/autoscaling/spike_traffic.yaml
```
11. Deploy HPA and compare deployment with latency.
```
kubectl apply -f k8s_manifests/datadog/frontend-hpa-latency.yaml
```

 
