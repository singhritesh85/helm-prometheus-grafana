Create EFS in AWS and EFS-provisioner in EKS then check kubectl get pods and kubectl get sc

#
```
NAME                               READY   STATUS    RESTARTS   AGE
efs-provisioner-5f96556dcc-dskvl   1/1     Running   0          77m

NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
aws-efs         example.com/aws-efs     Delete          Immediate              false                  78m
```

Installing Prometheus and Grafana

# Add and update repos to fetch charts
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

## Create monitoring namespace
`kubectl create namespace monitoring`

## Install Prometheus service and define storage
```
helm install prometheus prometheus-community/prometheus \
--namespace monitoring \
--set alertmanager.persistentVolume.storageClass="standard" \
--set server.persistentVolume.storageClass="standard"
```

## Create grafana.yaml
```
mkdir -p ${HOME}/environment/grafana

cat << EoF > ${HOME}/environment/grafana/grafana.yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.monitoring.svc.cluster.local
      access: proxy
      isDefault: true
EoF
```

## Install grafana and allocate storage and user pass
```
helm install grafana grafana/grafana \
    --namespace monitoring \
    --set persistence.storageClassName="standard" \
    --set persistence.enabled=true \
    --set adminPassword='subje' \
    --values ${HOME}/environment/grafana/grafana.yaml \
    --set service.type=LoadBalancer
```

## Import a predefined template within Grafana:
For example template ID: 6417 
https://grafana.com/grafana/dashboards/6417
