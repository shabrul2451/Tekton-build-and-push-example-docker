# To build and push the image to Docker Hub we have to follow the steps below:
- Create secrets and service account.
- Create [Pipeline Resources](https://github.com/tektoncd/pipeline/blob/main/docs/resources.md).
- Create [Task](https://github.com/tektoncd/pipeline/blob/main/docs/tasks.md).
- Create [Taskrun](https://github.com/tektoncd/pipeline/blob/main/docs/taskruns.md).
- Observe the task run status and logs.

### To create Secrets:
Registry Example:
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
Git example:
```json
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-username-password-git-secret
  annotations:
    tekton.dev/git-0: https://github.com/
type: kubernetes.io/basic-auth
stringData:
  username: <your-git-username>
  password: <your-git-password>
```
To apply:
```cmd
kubectl apply -f basic-auth-username-password-git-secret.yaml
```

[N:B: If git repository is public then you can ignore git secret.]

### To create Service Account:
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