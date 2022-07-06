# DEPLOY PROMETHEUS

## add prometheus Helm repo

`helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`

## add grafana Helm repo

`helm repo add grafana https://grafana.github.io/helm-charts`

## DEPLOY PROMETHEUS

`kubectl create namespace prometheus`

`helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`

```bash
helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```


access prometheus from localhost using port forwarding at http://localhost:8080

`kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090`

# DEPLOY GRAFANA

`mkdir -p ${HOME}/environment/grafana`

```bash
cat << EoF > ${HOME}/environment/grafana/grafana.yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true
EoF
```

`kubectl create namespace grafana`

```bash
helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='EKS!sAWSome' \
    --values ${HOME}/environment/grafana/grafana.yaml \
    --set service.type=LoadBalancer
```

`kubectl get all -n grafana`


after a few minutes after a classic LB is provisioned

```bash
export ELB=$(kubectl get svc -n grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

echo "http://$ELB"
```

---

# ADD CLUSTER AND PODS MONITORING DASHBOARDS

- CLUSTER
  - import dashboard
  - enter **3119** dashboard id
  - click **load**
  - select **Prometheus** as the endpoint under data sources dropdown
  - click **Import**
- PODS  
  - import dashboard
  - enter **6417** dashboard id
  - click **load**
  - select **Prometheus** as the endpoint under data sources dropdown
  - click **Import**


