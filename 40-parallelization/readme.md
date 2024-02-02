# Running tests in parallel on Kubernetes
## Pabot
[Pabot](https://pabot.org/) is the standard project for running Robot Framework tests in parallel. You can use Pabot in containers and also in containers running in Kubernetes. But it can only scale by threads in one container. This means you need to assign enough resources to one pod/container to be able to run a predicted amount of threads of Robot Framework. E.g. running 20 parallel Robot Framework tests with using the Requests Library needs roughly 4 vCPU and 4 GB of memory.

### configuring pod resources
[The resources would be defined as part of your pod definition.](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
```
    resources:
      requests:
        memory: "4000Mi"
        cpu: "4000m"
      limits:
        memory: "4000Mi"
        cpu: "4000m"
```
An important detail to keep in mind is the [Quality of Service of a pod](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/), which is based on its resource configuration.


## Scaling into multiple containers
It would be better to split the tests into several container to let the resource management of Kubernetes do the required scaling based on the amount of created pods. This can be achieved by [node autoscaling](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler) (which is different from cloud provider to cloud provider) or by [horizontal pod autoscaling](https://kubernetes.io/de/docs/tasks/run-application/horizontal-pod-autoscale/), if the pod pulls in dynamically the test it needs to execute. 

## Current development on this topic
There are currently to projects in development to provide solutions for better scaling of tests across multiple containers.
- [Probot](https://github.com/mwfoekens/Probot). [Merel, the creator of this project, will be presenting at RoboCon online](https://robocon.io/#online-probot-test-parallelization--distribution-with-docker-and-kubernetes). 
- [RobotFramework-Cluster](https://github.com/MarketSquare/robotframework-cluster), which is currently still in conception and is planned to be a successor of Pabot.

