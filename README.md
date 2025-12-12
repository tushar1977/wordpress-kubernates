## Building Image 

```bash
cd openresty
docker buildx build -t open .
cd ..
```

## To run wordpress

```bash
helm install wp wordpress_helm/
```
After that, you can visit ```http://localhost:30080```

## For Grafana and Prometheus

```bash
helm install grafana grafana/grafana
helm install prometheus prometheus-community/prometheus
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext
```

2) Now open http://localhost:30243
3) You can find grafana credudencials by ```kubectl get secret grafana -o jsonpath="{.data}"```
4) Add a new Data source in Grafana using that Prometheus
5) Import the ```15661``` dashboard


## About Directory

```.
├── .gitignore                  # Git ignore rules for Helm charts, Docker artifacts, and temp files.
├── .helmignore                 # Helm-specific ignores (e.g., .git, charts/ subdirs) to exclude from packaging.
├── openresty/                  # Docker build files.
│   ├── Dockerfile              # Multi-stage Dockerfile for OpenResty: Builds from source with custom ./configure, copies to runtime Ubuntu image.
│   └── openresty.conf          # Nginx/OpenResty config: Proxies to WordPress, serves static files, includes Lua block, and logs to stdout.
└── wordpress_helm/             # Helm chart directory for K8s deployment.
    ├── Chart.yaml                  # Helm chart metadata: Name (`wordpress`), version, description, and dependencies.
    └── values.yaml                 # Helm values: Customizable params like replicas, image tags, storage sizes, and secrets refs.
    ├── templates/               # Kubernetes manifests (templated YAMLs).
    │   ├── mysql/               # MySQL resources.
    │   │   ├── mysql-service.yaml     # Service exposing MySQL on port 3306 (ClusterIP).
    │   │   └── mysql-stateful.yaml    # StatefulSet for MySQL with PVC for /var/lib/mysql (RWO storage).
    │   |   ├── secrets.yaml          # Secret for DB credentials.
    │   ├── openresty/            # OpenResty resources.
    │   │   ├── configmap.yaml    # ConfigMap mounting openresty.conf to /opt/openresty/nginx/conf/.
    │   │   ├── deployment.yaml   # Deployment for OpenResty pods (replicas from values.yaml, mounts shared PVC read-only).
    │   │   └── service.yaml      # Using Nodeport for OpenResty on port 80.
    │   ├── persistentStorage/    # Storage resources.
    │   │   └── pv.yaml           # PersistentVolume for RWX.
    │   ├── wordpress/            # WordPress resources.
    │   │   ├── deployment.yaml   # Deployment for WordPress (PHP-FPM, env vars for DB, mounts RWX PVC at /var/www/html).
    │   │   └── service.yaml      # ClusterIP Service for WordPress on port 80 (internal proxy target).
    │   └── namespaces.yaml       # Namespace creation.
