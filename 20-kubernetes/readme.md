# Kubernetes Basics Workshop

## Requirements
- Access to a kubernetes cluster. This can be a local [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) cluster (kubernetes in docker) or a GKE cluster.
- For GKE, we need the [google cloud CLI](https://cloud.google.com/sdk/docs/install)
- And we need [kubectl](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl) to be installed

## Other useful resources
About YAML syntax: 
- https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html
- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Let's get started
### Kubeconfig and cluster connection
- What is my current kubectl context?
  - `kubectl config view --minify --flatten --context=$(kubectl config current-context)` or in short `kubectl config view | grep "current"`
  [Mastering the kube config file](https://ahmet.im/blog/mastering-kubeconfig/)

- Connect to our workshop k8s cluster
  - `gcloud container clusters get-credentials k8s-workshop-cluster --region europe-north1 --project robocon2024-workshop`

### Create an app
- Create your namespace
  - `kubectl create ns your-demo-ns`
- Export the namespace name as a variable
  - `export MY_NAMESPACE=your-demo-ns`

- Deploy an nginx container with one kubectl command
  - `kubectl create deployment nginx --image=nginx -n $MY_NAMESPACE`

- Have a look at the created deployment object in the cluster
  - `kubectl get deploy -n $MY_NAMESPACE`
  - `kubectl get deploy -n $MY_NAMESPACE -o wide` for more context
  - `kubectl get deploy -n $MY_NAMESPACE -o yaml` for full context

- Check the status of your pods
  - `kubectl get pods -n $MY_NAMESPACE`
  - `kubectl get pods -n $MY_NAMESPACE -o wide` for more context
  - `kubectl get pods -n $MY_NAMESPACE -o yaml` for full context

- Port forward to see the website
  - `export POD=$(kubectl get pods -n $MY_NAMESPACE -o name)`
  - `kubectl port-forward $POD 8080:80 -n $MY_NAMESPACE`
  - Access nginx in browser: [http://localhost:8080/](http://localhost:8080/)

## Let's customize that website
### How is this nginx configured?
- Execute into the pod
  - `kubectl exec -it $POD -n $MY_NAMESPACE -- bash`
- Explore the container
  - `cat etc/nginx/conf.d/default.conf`
  - `ls -la /usr/share/nginx/html`
    [Nginx documentation](http://nginx.org/en/docs/beginners_guide.html)
    [Nginx on Dockerhub](https://hub.docker.com/_/nginx)

### Editing index.html
- Installing vim in the container
  - `apt update`
  - `apt install vim`
  - `vim /usr/share/nginx/html/index.html`

- Check changes via port-forward and navigating to [http://localhost:8080/](http://localhost:8080/)

### Deleting the pod
- This will remove your changes as the pod will be re-created from the deployment definition
  - `kubectl delete pod $POD -n $MY_NAMESPACE`
  
## Inject the index.html via a configmap into the pod
### Create a configmap
Create a configmap based on your index.html
- `kubectl create configmap index-html --from-file=index.html -n $MY_NAMESPACE`

Check the config map
- kubectl get cm index-html -n $MY_NAMESPACE -o yaml

[Documentation on configuring pods with configmaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

### Inject the config map into the pod via a volume
[Documentation on using a configmap as a volume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume)

Get the kubernetes manifest of your current deployment
- `kubectl get deploy nginx -n $MY_NAMESPACE -o yaml > nginx-deployment.yaml`

Clean up the yaml from cluster specific content via this online tool
[kubernetes-manifest-cleaner](https://tools.tutorialworks.com/kubernetes-manifest-cleaner/)

Add the volume and the volumeMount to it
volume under spec:
```
      volumes:
      - name: index-html-vol
        configMap:
          name: index-html
          items:
          - key: index-html-file-content
            path: index.html
```

volumeMount under containers
```
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: index-html-vol
```

Apply the changes back to the cluster
- `kubectl replace -f nginx-deployment.yaml`

Check your pod definition containing your changes
- `kubectl get pods -n $MY_NAMESPACE -o yaml`