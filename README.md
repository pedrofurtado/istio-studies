# Istio studies

Istio service mesh studies. Just for fun.

```bash
# Enter root folder of repo (commands must be run in root folder)
cd istio-studies/

# Create k8s cluster
kind create cluster
kubectl cluster-info --context kind-kind

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

# Deploy app + Access the app at http://localhost:3010
kubectl apply -f k8s/
watch kubectl get pods
while true; do curl http://$(kubectl get nodes -o json | jq -r '.items[].status.addresses[] | select(.type=="InternalIP") | .address'):30001; sleep 0.5; done

# Delete the k8s cluster
kubectl delete all --all
kubectl delete "$(kubectl api-resources --namespaced=true --verbs=delete -o name | tr "\n" "," | sed -e 's/,$//')" --all
kubectl delete all --all --all-namespaces
kind delete cluster
```
