# Istio studies

Istio service mesh studies. Just for fun.

```bash
# Enter root folder of repo (commands must be run in root folder)
cd istio-studies/

# Install k3d
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | TAG=v5.6.3 bash
k3d --version

# Create k8s cluster
# Obs: The external port 8000 binds to node port 30000, that binds to the service port
k3d cluster create -p "8000:30000@loadbalancer" --agents 1
kubectl cluster-info

# Install IstioCTL + Create temporary alias for IstioCTL bin
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.21.2 TARGET_ARCH=x86_64 sh -
alias istioctl="$(pwd)/istio-1.21.2/bin/istioctl"
istioctl version

# Apply Istio to k8s cluster + Auto-inject sidecar proxy to every pod created in cluster
istioctl install
kubectl label namespace default istio-injection=enabled
watch kubectl get ns

# Apply the Istio add-ons
kubectl apply -f https://raw.githubusercontent.com/istio/istio/1.21.2/samples/addons/grafana.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/1.21.2/samples/addons/jaeger.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/1.21.2/samples/addons/kiali.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/1.21.2/samples/addons/loki.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/1.21.2/samples/addons/prometheus.yaml
watch kubectl get pods -n istio-system

# Access Kiali dashboard at http://localhost:3012
istioctl dashboard kiali --address 0.0.0.0 --port 3012 --browser=false

# Install fortio to simulate heavy loads
kubectl apply -f https://raw.githubusercontent.com/istio/istio/1.21.2/samples/httpbin/sample-client/fortio-deploy.yaml

# Deploy app + Access the app at http://localhost:8000
kubectl apply -f k8s/
watch kubectl get pods

# Simulate heavy loads
while true; do curl http://localhost:8000; sleep 0.5; done
or
kubectl exec "$(kubectl get pods -l app=fortio -o 'jsonpath={.items[0].metadata.name}')" -c fortio -- /usr/bin/fortio load -c 2 -qps 0 -t 200s -loglevel Warning http://nginx-service:8085

# Delete the k8s cluster
kubectl delete all --all
kubectl delete "$(kubectl api-resources --namespaced=true --verbs=delete -o name | tr "\n" "," | sed -e 's/,$//')" --all
kubectl delete all --all --all-namespaces
kind delete cluster
```
