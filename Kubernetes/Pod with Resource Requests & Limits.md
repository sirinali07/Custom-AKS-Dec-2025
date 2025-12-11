

### Create a Pod with resource requests & limits

In Kubernetes, you can explicitly assign CPU and Memory resources to a container to control how much minimum and maximum compute it gets.

**Requests =** Minimum CPU and Memory the Pod needs to run.
Kubernetes uses this to decide which node can host the Pod.

**Limits =** Maximum CPU and Memory the Pod is allowed to use.

If the Pod tries to use more:
* CPU → throttled (slowed down)
* Memory → Pod may get OOMKilled

**Reason to set resources:**

- Prevent one Pod from consuming too many resources
- Ensure fair usage
- Improve performance and stability
- Help scheduler place Pods correctly

---

Let's explicitly assign CPU and memory.

Create a YAML file 
```bash
vi pod-with-resources.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-resources
spec:
  containers:
    - name: nginx-ctr
      image: nginx
      ports:
        - containerPort: 80

      # ------------------------------
      # Resource Configuration Section
      # ------------------------------
      resources:
        requests:
          cpu: "100m"      # Minimum CPU required (0.1 core)
          memory: "128Mi"  # Minimum memory required
        limits:
          cpu: "200m"      # Maximum CPU allowed (0.2 core)
          memory: "256Mi"  # Maximum memory allowed

```
save the file using `ESCAPE + :wq!`

Apply the YAML
```bash
kubectl apply -f pod-with-resources.yaml
```
Check Pod status

```bash
kubectl get pods
```
Describe the Pod

```bash
kubectl describe pod nginx-with-resources
```

