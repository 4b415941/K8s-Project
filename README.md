##k8s project at https://labs.play-with-k8s.com/

## Cluster Setup

1. **Cluster Initialization**: Set up a Kubernetes cluster with one master and four worker nodes.
    ```bash
    $ kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16
    ```

2. **Networking Initialization**: Initialize cluster networking using kube-router for pod networking.
    ```bash
    $ kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
    $ export KUBECONFIG=/etc/kubernetes/admin.conf
    $ kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/master/manifests/calico.yaml
    ```

3. **Join Worker Nodes**: Add the four worker nodes to the cluster.
    ```bash
    $ kubeadm join 192.168.0.23:6443 --token c4vy8w.nnkgmvgeu1v68vqh --discovery-token-ca-cert-hash sha256:479dfae3ba09cf4c2193649bbd91f70636ba8db63ef8de91239d1a3ba063cca6 
    ```

## Namespace and RBAC Configuration

4. **Create Namespaces**: Create "test" and "production" namespaces.
    ```bash
    $ kubectl create namespace test
    $ kubectl create namespace production
    ```

5. **Role Creation and Association**: Create roles for "junior" and "senior" groups with specific permissions and associate them with the respective namespaces.
    ```bash
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/k8s-Project/main/jr-test-rb.yaml
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/k8s-Project/main/jr-production-rb.yaml
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/K8s-Project/main/sr-cluster-crb.yaml
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/K8s-Project/main/sr-production-rb.yaml
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/K8s-Project/main/sr-test-rb.yaml
    ```

## Ingress Controller Installation

6. **Install Ingress Controller**: Deploy the Nginx Ingress Controller.
    ```bash
    $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/cloud/deploy.yaml
    ```

## Node Configuration

7. **Node Taints and Labels**: Configure three worker nodes to only allow deployment of production pods and label them accordingly.
    ```bash
    $ kubectl taint node node1 tier=production:NoSchedule
    $ kubectl taint node node2 tier=production:NoSchedule
    $ kubectl taint node node3 tier=production:NoSchedule
    $ kubectl label node node1 tier=production
    $ kubectl label node node2 tier=production
    $ kubectl label node node3 tier=production
    ```

## Application Deployment

8. **Deploy WordPress**: Deploy WordPress in both "test" and "production" namespaces.
    ```bash
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/k8s-Project/main/wptest.yaml
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/k8s-Project/main/wpprod.yaml
    ```

9. **Expose WordPress**: Expose WordPress deployments through Ingress with different hostnames.
    ```bash
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/k8s-Project/main/wpingress.yaml
    ```

10. **Deployment with Update Strategy**: Create a deployment in the "production" namespace with rolling updates and health checks.
    ```bash
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/k8s-Project/main/deployment.yaml
    ```

11. **Expose Deployment**: Expose the deployment with a LoadBalancer service.
    ```bash
    $ kubectl expose deployment k8s-deployment --type=LoadBalancer -n production
    ```

## Scaling and Updating

12. **Scale Deployment**: Scale the deployment from 3 to 10 replicas.
    ```bash
    $ kubectl scale deployment k8s-deployment --replicas=3 -n production
    $ kubectl scale deployment k8s-deployment --replicas=10 -n production
    ```

13. **Update Deployment**: Update the deployment with a new image.
    ```bash
    $ kubectl set image deployment/k8s-deployment k8s=ozgurozturknet/k8s:v2 -n production
    ```

## Monitoring

14. **Monitoring Setup**: Install Prometheus and Grafana for cluster monitoring.
    ```bash
    $ kubectl create namespace monitoring
    $ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    $ chmod 700 get_helm.sh
    $ ./get_helm.sh
    $ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    $ helm repo update
    $ helm install kubeprostack --namespace monitoring prometheus-community/kube-prometheus-stack
    ```

15. **Access Grafana**: Forward port to access Grafana dashboard.
    ```bash
    $ kubectl --namespace monitoring port-forward svc/kubeprostack-grafana 8080:80
    ```

## Additional Components

16. **Fluentd Deployment**: Deploy Fluentd as a DaemonSet for logging.
    ```bash
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/k8s-Project/main/daemonset.yaml
    ```

17. **MongoDB StatefulSet**: Deploy a MongoDB cluster as a StatefulSet.
    ```bash
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/k8s-Project/main/statefulset.yaml
    ```

## Service Account and Pod Interaction

18. **Create Service Account**: Create a service account with read permissions across all objects.
    ```bash
    $ kubectl apply -f https://raw.githubusercontent.com/4b415941/k8s-Project/main/serviceaccount.yaml
    ```

19. **Interact with Cluster**: Connect to a pod using the service account and list all pods.
    ```bash
    $ kubectl exec -it pod-proje -- bash
    ```

