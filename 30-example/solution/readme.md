# Solution for containerizing the Grafana test

## Dockerfile
The Docker image is based on [ppodgorsek/robot-framework](https://github.com/ppodgorsek/docker-robot-framework) as it fulfills most of our requirements.
In this image [the user is set to a default user](https://github.com/ppodgorsek/docker-robot-framework/blob/master/Dockerfile#L160) running the test. To be able to install on top of what is already existing, we need to set the `USER root`.
We then copy the test into the [tests folder of the image](https://github.com/ppodgorsek/docker-robot-framework/blob/2b470521c246bb5e12404d744da4d6849b7d683b/Dockerfile#L10).
We replace the original [run-tests-in-virtual-screen.sh](https://github.com/ppodgorsek/docker-robot-framework/blob/master/bin/run-tests-in-virtual-screen.sh) with a slightly modifided version. I added here a while loop in the end to make sure the container stays alive to be able to serve the test report.
Finally we install the KubeLibrary.

## Service account permissions inside the cluster
To be able to read the Kuberentes resources from the grafana namespace in the cluster, we need to run the tests with a service account having elevated user rights. [k8s-sa.yml](k8s-sa.yml)

## Setting the kube context for the KubeLibrary
The KubeLibrary uses by the default the current kube context from your kube_config file. For incluster execution we need to pass on the parameter `incluster=True`.

When running the KubeLibrary locally in a Docker container, you'll need to share your kube config via a volume with the container. [It is explained here.](https://stackoverflow.com/questions/67511646/how-to-pass-in-kubernetes-config-into-a-docker-run-command)

## Kubernetes manifest
The Kubernetes manifest is in this case a [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/) as we only want to run the test once.
We run the image, which we built from the Dockerfile and add an [nginx](https://nginx.org/en/) container as a [sidecar](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) to access a rendered version of the test report.
The test report is shared via an emptydir volume, between the containers.


## Step by step execution
- Build the docker image
`docker build -t my-image-name .`

- Get the image id
`docker image list`

- Tag the image
`docker tag <image-id> registry.humanitec.io/k8swrkshp/my-image-name:0.0.1`

- Upload the image to the registry
`docker push registry.humanitec.io/k8swrkshp/my-image-name:0.0.1`

- Update the image name in the [job.yml](job.yml) file.

- Connect to the kubernetes cluster
`gcloud container clusters get-credentials k8s-workshop-cluster --region europe-north1 --project robocon2024-workshop`

- Setting your namespace as an environment variable
`export MY_NAMESPACE=my-namespace-name`

- Create your namespace
`kubectl create ns ${MY_NAMESPACE}`

- Setting Docker registry credentials
When the container image needs to be pulled from a private registry, the Kubernetes cluster needs access credentials. These are provided in a secret. The `regcred` secret can be created via this command:
`kubectl create secret docker-registry regcred --docker-server=registry.humanitec.io --docker-username=k8swrkshp --docker-password=${REGISTRY_PASSWORD} --docker-email=nils@humanitec.com -n ${MY_NAMESPACE}`

- Setting service account permissions
`kubectl apply -f k8s-sa.yml -n ${MY_NAMESPACE}`

- Adding the ClusterRoleBinding
  ```
  kubectl apply -f - <<EOF
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: tester
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: cluster-admin
  subjects:
  - kind: ServiceAccount
    namespace: '${MY_NAMESPACE}'
    name: tester
  EOF
  ```

- Trigger the robot-test job
`kubectl apply -f job.yml -n ${MY_NAMESPACE}`

- Check the running pods triggered by the job
`kubectl get pods -n ${MY_NAMESPACE}`

- Export the pod name into an environment variable
`export POD=$(kubectl get pods -n $MY_NAMESPACE -o name)`

- Check the container logs
`kubectl logs ${POD} -c robot-run -n ${MY_NAMESPACE}`

- Port forward to the nginx container to see the test report
`kubectl port-forward ${POD} 8080:80 -n ${MY_NAMESPACE}`
Navigate in your browser to http://localhost:8080/report.html


# Optional
## Exposing your test report via a public DNS record
In this example, I'm using a DNS record, which is managed in an AWS route 53 account. I configured the record *.k8swrkshp.devopsplatforms.tv to route traffic to the [ingress-nginx](https://github.com/kubernetes/ingress-nginx) loadbalancer inside the cluster. In addition, I set up [cert manager](https://cert-manager.io/) provide me an up to date certificate for the domain.

- First, you'll need to define a [service object](https://kubernetes.io/docs/concepts/services-networking/service/) pointing to the port of the nginx container. [service.yml](service.yml)
- Second, you'll need a [tls certificate secret object](https://kubernetes.io/docs/concepts/configuration/secret/#tls-secrets) to provide a valid tls certificate. We'll use here the powers of the cert manager by providing a [certificate custom resource definition](https://cert-manager.io/docs/usage/certificate/). [certificate-crd.yml](certificate-crd.yml)
- Third, You'll need an [ingress object](https://kubernetes.io/docs/concepts/services-networking/ingress/) binding a DNS record in the ingress-nginx configuration to the service object. [ingress.yml](ingress.yml) Since this ingress is publishing the test report to a public domain, it could be good to some authentification in front of it. For this I added annotations to the ingress object, which trigger the ingress controller to ask for a basic authentification with username and password. [This is explained here.](https://kubernetes.github.io/ingress-nginx/examples/auth/basic/)

## Step by step execution
- Create the service object pointing to port 80 of the nginx container
`kubectl apply -f service.yml -n ${MY_NAMESPACE}`
- Update the subdomain in [ingress.yml](ingress.yml) and [certificate-crd.yml](certificate-crd.yml). e.g. test-report.k8swrkshp.devopsplatforms.tv. Keep in mind that domain names need to be globally unique.
- Create the certificate CRD, which will provide the tls certificate secret
`kubectl apply -f certificate-crd.yml -n ${MY_NAMESPACE}`
- Create a htpasswd file
  ```
  htpasswd -c auth tester
  New password: <bar>
  New password:
  Re-type new password:
  Adding password for user tester
  ```
- Create the secret object for the basic authentication
`kubectl create secret generic basic-auth --from-file=auth -n ${MY_NAMESPACE}`
- Create the ingress object
`kubectl apply -f ingress.yml -n ${MY_NAMESPACE}`

