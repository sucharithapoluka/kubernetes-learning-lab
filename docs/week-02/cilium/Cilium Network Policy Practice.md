Cilium Network Policy Practice
Part 1: Create clean kind cluster for Cilium

Use a kind cluster with default CNI disabled and kube-proxy disabled.

1. Create config file
cat <<EOF > kind.yml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  kubeProxyMode: "none"
nodes:
- role: control-plane
EOF
2. Delete old cluster if present
kind delete cluster --name kind
3. Create new cluster
kind create cluster --config kind.yml
4. Check cluster
kubectl get nodes
kubectl get pods -A
Part 2: Install Cilium
1. Install Cilium CLI if not already installed
curl -L --remote-name https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz
sudo tar -xvf cilium-linux-amd64.tar.gz -C /usr/local/bin
cilium version
2. Install Cilium in cluster
cilium install
3. Verify
cilium status
kubectl get pods -n kube-system
Part 3: Enable Hubble
1. Enable Hubble UI
cilium hubble enable --ui
2. Verify again
cilium status
kubectl get pods -n kube-system
kubectl get svc -n kube-system
3. Port-forward Hubble UI
kubectl -n kube-system port-forward svc/hubble-ui 12000:80

If remote VM and you want browser access from outside:

kubectl -n kube-system port-forward --address 0.0.0.0 svc/hubble-ui 12000:80
Part 4: Deploy demo application

We will create:

backend = nginx server

frontend = busybox client

1. Create namespace
kubectl create ns demo
2. Create backend deployment
kubectl -n demo create deployment backend --image=nginx
3. Expose backend as service
kubectl -n demo expose deployment backend --port=80 --name=backend-svc
4. Create frontend deployment
kubectl -n demo create deployment frontend --image=busybox -- sleep 3600
5. Check deployments and pods
kubectl get deploy -n demo
kubectl get pods -n demo -o wide
kubectl get svc -n demo
6. Label deployments properly
kubectl -n demo label deploy/frontend app=frontend --overwrite
kubectl -n demo label deploy/backend app=backend --overwrite
7. Confirm labels
kubectl get deploy -n demo --show-labels
kubectl get pods -n demo --show-labels
Part 5: Test baseline connectivity

Before policy, frontend should reach backend.

kubectl -n demo exec -it deploy/frontend -- wget -qO- http://backend-svc

If working, you should see nginx HTML output.

You can also open shell inside frontend:

kubectl -n demo exec -it deploy/frontend -- sh

Then run inside pod:

wget -qO- http://backend-svc

Exit:

exit
Part 6: Apply DENY policy

This policy blocks frontend -> backend traffic.

1. Create deny policy file
cat <<EOF > deny-egress.yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: deny-frontend-to-backend
  namespace: demo
spec:
  endpointSelector:
    matchLabels:
      app: frontend
  egressDeny:
  - toEndpoints:
    - matchLabels:
        app: backend
EOF
2. Apply deny policy
kubectl apply -f deny-egress.yaml
3. Verify policy
kubectl get ciliumnetworkpolicy -n demo
kubectl describe ciliumnetworkpolicy deny-frontend-to-backend -n demo
4. Test again
kubectl -n demo exec -it deploy/frontend -- wget -qO- --timeout=3 http://backend-svc || echo "Denied!"

Expected result:

Denied!
Part 7: Observe dropped traffic in Hubble

Open another terminal and run:

hubble observe --verdict DROPPED

Then again generate traffic:

kubectl -n demo exec -it deploy/frontend -- wget -qO- --timeout=3 http://backend-svc || echo "Denied!"

You should see dropped flow in Hubble.

Part 8: Remove deny policy

Before testing allow, delete deny policy first.

kubectl delete ciliumnetworkpolicy deny-frontend-to-backend -n demo

Check:

kubectl get ciliumnetworkpolicy -n demo

Test connectivity again:

kubectl -n demo exec -it deploy/frontend -- wget -qO- http://backend-svc

It should work again.

Part 9: Apply ALLOW policy

Your earlier allow policy used labels server and client, but your actual deployments are backend and frontend.

So use this corrected policy.

1. Create allow policy file
cat <<EOF > allow.yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: demo
spec:
  endpointSelector:
    matchLabels:
      app: backend
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
EOF
2. Apply allow policy
kubectl apply -f allow.yaml
3. Verify
kubectl get ciliumnetworkpolicy -n demo
kubectl describe ciliumnetworkpolicy allow-frontend-to-backend -n demo
4. Test traffic
kubectl -n demo exec -it deploy/frontend -- wget -qO- http://backend-svc

It should work.

Part 10: Important understanding
Deny policy meaning
endpointSelector:
  matchLabels:
    app: frontend
egressDeny:
- toEndpoints:
  - matchLabels:
      app: backend

Meaning:

select pod = frontend

block its outgoing traffic

destination = backend

So:

frontend -> backend = blocked

Allow policy meaning
endpointSelector:
  matchLabels:
    app: backend
ingress:
- fromEndpoints:
  - matchLabels:
      app: frontend

Meaning:

select pod = backend

allow incoming traffic

source = frontend

So:

frontend -> backend = allowed

Part 11: Useful commands for practice
Check pods
kubectl get pods -n demo -o wide
Check services
kubectl get svc -n demo
Check endpoints
kubectl get endpoints -n demo
Check Cilium policies
kubectl get ciliumnetworkpolicy -n demo
Describe one policy
kubectl describe ciliumnetworkpolicy allow-frontend-to-backend -n demo
Enter frontend pod shell
kubectl -n demo exec -it deploy/frontend -- sh
Test DNS inside pod
kubectl -n demo exec -it deploy/frontend -- nslookup backend-svc
Test HTTP
kubectl -n demo exec -it deploy/frontend -- wget -qO- http://backend-svc
Watch dropped traffic
hubble observe --verdict DROPPED
Watch all flows
hubble observe
Part 12: Full quick practice version

If you want only the exact command sequence:

kind delete cluster --name kind

cat <<EOF > kind.yml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  kubeProxyMode: "none"
nodes:
- role: control-plane
EOF

kind create cluster --config kind.yml

cilium install
cilium status

cilium hubble enable --ui
kubectl -n kube-system port-forward svc/hubble-ui 12000:80

kubectl create ns demo
kubectl -n demo create deployment backend --image=nginx
kubectl -n demo expose deployment backend --port=80 --name=backend-svc
kubectl -n demo create deployment frontend --image=busybox -- sleep 3600

kubectl -n demo label deploy/frontend app=frontend --overwrite
kubectl -n demo label deploy/backend app=backend --overwrite

kubectl get pods -n demo --show-labels
kubectl get svc -n demo

kubectl -n demo exec -it deploy/frontend -- wget -qO- http://backend-svc

Create deny policy:

cat <<EOF > deny-egress.yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: deny-frontend-to-backend
  namespace: demo
spec:
  endpointSelector:
    matchLabels:
      app: frontend
  egressDeny:
  - toEndpoints:
    - matchLabels:
        app: backend
EOF

Apply and test:

kubectl apply -f deny-egress.yaml
kubectl get ciliumnetworkpolicy -n demo
kubectl -n demo exec -it deploy/frontend -- wget -qO- --timeout=3 http://backend-svc || echo "Denied!"

Delete deny:

kubectl delete ciliumnetworkpolicy deny-frontend-to-backend -n demo
kubectl -n demo exec -it deploy/frontend -- wget -qO- http://backend-svc

Create allow policy:

cat <<EOF > allow.yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: demo
spec:
  endpointSelector:
    matchLabels:
      app: backend
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
EOF

Apply and test:

kubectl apply -f allow.yaml
kubectl get ciliumnetworkpolicy -n demo
kubectl -n demo exec -it deploy/frontend -- wget -qO- http://backend-svc