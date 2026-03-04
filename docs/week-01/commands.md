# Kubernetes & Kind Commands Reference

| Category | Command | Description |
|--------|--------|-------------|
| Tool Version | `kubectl version` | Shows Kubernetes client and server version |
| Tool Version | `kind --version` | Shows Kind version |
| Tool Version | `helm version` | Shows Helm version |
| Tool Version | `terraform --version` | Shows Terraform version |
| Tool Version | `k9s version` | Shows k9s version |
| Cluster | `kind create cluster` | Creates default single-node cluster |
| Cluster | `kind create cluster --name dev-cluster` | Creates cluster with custom name |
| Cluster | `kind create cluster --config kind-config.yaml` | Creates multi-node or custom cluster |
| Cluster | `kind get clusters` | Lists all Kind clusters |
| Cluster | `kind delete cluster` | Deletes default cluster |
| Cluster | `kind delete cluster --name dev-cluster` | Deletes specific cluster |
| Cluster | `kubectl cluster-info` | Shows cluster control-plane endpoints |
| Config | `kubectl config get-contexts` | Shows configured cluster contexts |
| Config | `kubectl config current-context` | Shows current active cluster |
| Config | `kubectl config use-context <context>` | Switches cluster context |
| Config | `cat ~/.kube/config` | Displays kubeconfig file |
| Nodes | `kubectl get nodes` | Lists nodes in cluster |
| Nodes | `kubectl get nodes -o wide` | Shows node details (IP, OS, runtime) |
| Nodes | `kubectl describe node <node>` | Shows detailed node information |
| Nodes | `kubectl top nodes` | Shows node CPU and memory usage |
| Pods | `kubectl get pods` | Lists pods in current namespace |
| Pods | `kubectl get pods -A` | Lists pods across all namespaces |
| Pods | `kubectl get pods -n <ns>` | Lists pods in specific namespace |
| Pods | `kubectl describe pod <pod> -n <ns>` | Shows pod details and events |
| Pods | `kubectl logs <pod> -n <ns>` | Displays pod logs |
| Pods | `kubectl logs -f <pod> -n <ns>` | Streams pod logs live |
| Pods | `kubectl logs --previous <pod>` | Shows previous container logs |
| Pods | `kubectl exec -it <pod> -- sh` | Opens shell inside container |
| Services | `kubectl get svc` | Lists services |
| Services | `kubectl get svc -A` | Lists services in all namespaces |
| Control Plane | `kubectl get pods -n kube-system` | Lists control-plane components |
| Control Plane | `kubectl logs -n kube-system <kube-apiserver>` | Shows API server logs |
| Control Plane | `kubectl logs -n kube-system <kube-scheduler>` | Shows scheduler logs |
| Control Plane | `kubectl logs -n kube-system <kube-controller-manager>` | Shows controller manager logs |
| Control Plane | `kubectl logs -n kube-system <etcd>` | Shows etcd logs |
| Docker | `docker ps` | Shows running containers (Kind nodes) |
| Docker | `docker exec -it kind-control-plane bash` | Access control-plane container |
| Filesystem | `cd /etc/kubernetes/manifests` | View static pod manifests |
| Logs | `kind export logs` | Export cluster logs |
| Logs | `kind export logs --name dev-cluster` | Export logs for specific cluster |
| Images | `kind load docker-image my-image:latest` | Load local image into cluster |
| Debugging | `journalctl -u kubelet -n 200` | View recent kubelet logs |
| Debugging | `journalctl -u kubelet -f` | Stream kubelet logs |
| Cleanup | `for c in $(kind get clusters); do kind delete cluster --name $c; done` | Delete all Kind clusters |