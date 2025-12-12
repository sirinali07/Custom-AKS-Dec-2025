## LAB GUIDE: Exposing a Frontend Application Using Gateway API (NGINX Gateway Fabric)

Deploy Gateway API components using NGINX Gateway Fabric, create a test application, and expose it using an HTTPRoute.



#### Step 1: Install Gateway API Resources

First, we need to install the Gateway API CRDs (Custom Resource Definitions):

**Install Gateway API resources**
```bash
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.5.1" | kubectl apply -f -
```

Verify installations
```bash
kubectl get crd | grep gateway
```

#### Step 2: Configure NGINX Gateway Fabric

Next, we'll install NGINX Gateway Fabric as our Gateway API controller:

**Deploy NGINX Gateway Fabric CRDs**
```
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/crds.yaml
```

**Deploy NGINX Gateway Fabric**
```
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/nodeport/deploy.yaml
```

Verify the deployment
```
kubectl get pods -n nginx-gateway
```

Now, let's configure specific ports for the gateway service:

View the nginx-gateway service
```
kubectl get svc -n nginx-gateway nginx-gateway -o yaml
```
***You can either patch NodePorts OR change service type → LoadBalancer.***

**Option 1: Update the service to specific NodePort values**
```
kubectl patch svc nginx-gateway -n nginx-gateway --type='json' -p='[
  {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30080},
  {"op": "replace", "path": "/spec/ports/1/nodePort", "value": 30081}
]'
```
### OR

**Option 2: Change the service type to LoadBalancer**
```
kubectl edit svc -n nginx-gateway nginx-gateway  
```
Change type of service from NodePort to LoadBalancer
> `type:``LoadBalancer`

Verify the updated service  
```
kubectl get svc -n nginx-gateway nginx-gateway
```

#### Step 3: Create GatewayClass and Gateway Resources

Now, let's create the GatewayClass and Gateway resources. Save the following YAML to a file named `gateway-resources.yaml`:
```
vi gateway-resources.yaml
```
```yaml
# GatewayClass defines the controller that implements the Gateway API
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: gateway.nginx.org/nginx-gateway-controller
---
# Gateway resource defines the listener and ports
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
    allowedRoutes:
      namespaces:
        from: All
```
save the file using `ESCAPE + :wq!`

Apply the configuration:
```
kubectl apply -f gateway-resources.yaml
```
Verify resources
```
kubectl get gatewayclass
```
```
kubectl get gateway
```
You should see both resources created

#### Step 4: Create a pod named frontend-app that exposes container on port 80
```
kubectl run frontend-app --port 80 --image nginx
```

#### Step 5: Create a svc that expose the frontend-app on port 80 with target port 80
```
kubectl expose pod frontend-app --port 80 --name frontend-svc
```

#### Step 6: Expose the Frontend Service with HTTPRoute
Now, let's create an HTTPRoute to route traffic to the frontend-svc service. Save the following YAML to frontend-route.yaml:
```
vi frontend-route.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: frontend-route
  namespace: default
spec:
  parentRefs:
  - name: nginx-gateway
    namespace: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: frontend-svc
      port: 80
```
save the file using `ESCAPE + :wq!`

Apply the configuration:
```
kubectl apply -f frontend-route.yaml
```
Verify the HTTPRoute
```
kubectl get httproute frontend-route
```
```
kubectl describe httproute frontend-route
```

The describe command should show that the HTTPRoute is accepted by the Gateway.

#### Step 5: Test the Basic Configuration
To test if your configuration works:

Get the Service External IP paste into the browser
 
> http://4.157.65.251/
 














