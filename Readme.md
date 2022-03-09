# To build and push the image to Docker Hub we have to follow the steps below:
- Create secrets and service account.
- Create pipeline resources.
- Create task.
- Create taskrun.
- Observe the task run status and logs.

### To create Secrets:
Example:
```json
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-username-password
  annotations:
    tekton.dev/docker-0: https://index.docker.io/v1/
type: kubernetes.io/basic-auth
stringData:
  username: <your_username>
  password: <your_password>
```
To apply:
```cmd
kubectl apply -f basic-auth-username-password.yaml
```

### To create Secrets:
Example:
```json
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-sa
secrets:
  - name: basic-auth-username-password
```
To apply:
```cmd
kubectl apply -f tekton-sa.yaml
```