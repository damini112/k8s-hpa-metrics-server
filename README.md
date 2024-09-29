# Kubernetes HPA with Metrics Server on kind Cluster

This repository contains the setup for Horizontal Pod Autoscaler (HPA) using Metrics Server on a kind Kubernetes cluster.

## Prerequisites

- A running kind cluster.
- `kubectl` installed and configured.

## Setup Instructions

### 1. Install Metrics Server

Download and apply the Metrics Server manifest:
<pre>
<code id="command">kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml</code>
</pre>

### 2. Patch the Metrics Server Deployment
Create a patch file named [metrics-server-patch.yaml](https://github.com/damini112/k8s-hpa-metrics-server/blob/main/metrics-server-patch.yaml) and apply the patch:
<pre>
  <code id="command">
  kubectl patch deployment metrics-server -n kube-system --patch "$(cat metrics-server-patch.yaml)"
  kubectl rollout restart deployment metrics-server -n kube-system
  </code>
</pre>
**Note:** Make sure the Metrics Server is up and running

### 3. Create a Deployment

Create a deployment using an NGINX container and save it to [nginx-deployment.yaml](https://github.com/damini112/k8s-hpa-metrics-server/blob/main/nginx-deployment.yaml) :

<pre>
  <code id="command">
  kubectl apply -f nginx-deployment.yaml
  </code>
</pre>

### 4. Expose the Deployment as a Service

Expose your deployment to make it accessible:
<pre>
  <code id="command">
kubectl expose deployment nginx-deployment --type=LoadBalancer --name=nginx-service --port=80 --target-port=80 
  </code>
</pre>

### 5. Create the HPA

Generate the HPA configuration and save it to hpa.yaml:
<pre>
<code id="command">
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=1 --max=10 -o yaml > hpa.yaml
</code>
</pre>


**Apply the HPA configuration:**
<pre>
  <code id="command">
kubectl apply -f hpa.yaml
  </code>
</pre>

### 6. Generate Load to Test the HPA

To test the HPA, generate load on your application:
<pre>
  <code id="command">
kubectl run -i --tty load-generator --image=busybox /bin/sh
  </code>
</pre>

**Inside the pod, run a loop to generate load:**
<pre>
<code id="command">
while true; do wget -q -O- http://nginx-service; done
</code>
</pre>


### Monitoring and Verification

Check HPA Status:
<pre>
<code id="command">
kubectl get deployment metrics-server -n kube-system
</code>
</pre>
<pre>
  <code id="command">
kubectl get hpa
  </code>
</pre>


### Check Deployment Status:
<pre>
<code id="command">
kubectl get deployment metrics-server -n kube-system
</code>
</pre>


### Troubleshooting

If the Metrics Server is not working, it might be related to port 443. Change it to port 8443 as shown in the patch file above. This change is necessary because binding to port 443 requires elevated permissions, which can cause issues in some environments.

### Conclusion

This setup ensures that the Metrics Server can handle secure communication while avoiding permission issues associated with binding to port 443.





