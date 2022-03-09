# To build and push the image to Dockerhub we have to follow the steps below:

- Create secrets and service account.
- Create [Pipeline Resources](https://github.com/tektoncd/pipeline/blob/main/docs/resources.md).
- Create [Task](https://github.com/tektoncd/pipeline/blob/main/docs/tasks.md).
- Create [Taskrun](https://github.com/tektoncd/pipeline/blob/main/docs/taskruns.md).
- Observe the task run status and logs.

### To create Secrets:

Registry Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-username-password-docker-secret
  annotations:
    tekton.dev/docker-0: https://index.docker.io/v1/
type: kubernetes.io/basic-auth
stringData:
  username: <your-registry-username>
  password: <your-registry-password>
```

Apply:

```cmd
kubectl apply -f basic-auth-username-password-docker-secret.yaml
```

Git example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-username-password-git-secret
annotations:
  tekton.dev/git-0:
    https: //github.com/
type: kubernetes.io/basic-auth
stringData:
username: <your-git-username>
password: <your-git-password>
```

Apply:

```cmd
kubectl apply -f basic-auth-username-password-git-secret.yaml
```

[N:B: If git repository is public then you can ignore git secret.]

### To create Service Account:

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-sa
secrets:
  - name: basic-auth-username-password
```

Apply:

```cmd
kubectl apply -f tekton-sa.yaml
```

### To create pipeline resource:

Pipeline resource Example(registry):

```yaml
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: docker-resource
spec:
  type: image
params:
  - name: url
value:
  shabrul2451/tekton-demo: 0.0.1
```

Apply:

```cmd
kubectl apply -f docker-resource.yaml
```

Pipeline resource Example(git):

```yaml
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: git-resource
spec:
  type: git
params:
  - name: revision
    value: master
  - name: url
    value: https://github.com/shabrul2451/test-app
```

Apply:

```cmd
kubectl apply -f git-resource.yaml
```

### To create Task:

Example:

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: build-docker-image-from-git-source
spec:
  params:
    - default: /workspace/docker-source/Dockerfile
      description: The path to the dockerfile to build
      name: pathToDockerFile
      type: string
    - default: /workspace/docker-source
      description: The build context used by Kaniko (https://github.com/GoogleContainerTools/kaniko#kaniko-build-contexts)
      name: pathToContext
      type: string
  resources:
    inputs:
      - name: docker-source
        type: git
    outputs:
      - name: builtImage
        type: image
  steps:
    - args:
        - --dockerfile=$(inputs.params.pathToDockerFile)
        - --destination=$(outputs.resources.builtImage.url)
        - --context=$(inputs.params.pathToContext)
      command:
        - /kaniko/executor
      env:
        - name: DOCKER_CONFIG
          value: /tekton/home/.docker/
      image: gcr.io/kaniko-project/executor:v1.6.0
      name: build-and-push
      resources: { }
```

Apply:

```cmd
kubectl apply -f build-docker-image-from-git-source.yaml
```

### To create TaskRun:

```yaml
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: build-docker-image-from-git-source-task-run
spec:
  serviceAccountName: tekton-sa
  taskRef:
    name: build-docker-image-from-git-source
  params:
    - name: pathToDockerFile
      value: Dockerfile
    - name: pathToContext
      value: /workspace/docker-source #configure: may change according to your source
  resources:
    inputs:
      - name: docker-source
        resourceRef:
          name: git-resource
    outputs:
      - name: builtImage
        resourceRef:
          name: docker-resource
```

Apply:

```cmd
kubectl apply -f build-docker-image-from-git-source-task-run.yaml
```

### To observe the taskrun status and logs:
```cmd
tkn taskrun describe build-docker-image-from-git-source-task-run
```
It will show the taskrun status and logs.
Example:
```cmd
Name:        build-docker-image-from-git-source-task-run
Namespace:   default
Task Ref:    build-docker-image-from-git-source
Service Account:   tekton-sa

🌡️  Status

STARTED          DURATION     STATUS
19 minutes ago   21 seconds   Succeeded

📨 Input Resources

No resources

📡 Output Resources

No resources

⚓ Params

No params

🦶 Steps

 NAME                               STATUS
 ∙ create-dir-builtimage-4255b      Completed
 ∙ git-source-docker-source-g48mk   Completed
 ∙ build-and-push                   Completed
 ∙ image-digest-exporter-sqb44      Completed

🚗 Sidecars

No sidecars

```

