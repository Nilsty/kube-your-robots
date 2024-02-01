# Workshop example test

This is a Robotframework example test featuring the [Browser Library](https://robotframework-browser.org/) and the [KubeLibrary](https://github.com/devopsspiral/KubeLibrary).

The test itself fetches details about a Grafana installation from a Kubernetes cluster to then log into the Grafana Web UI.

# Your task

Containerize this test by providing a dockerfile for it and execute it inside the Kubernetes cluster.

# Running locally

To run this test locally you will need to install
- Node.js (requirement of the Browser Library) [https://nodejs.org/en/download](https://nodejs.org/en/download)
- (Recommended) Use a python virtual environment e.g. [virtualenv](https://virtualenv.pypa.io/en/latest/)
- Install the python requirements via `pip install -r requirements.txt` 
- Initialize the Browser Library via `rfbrowser init`

And you'll need to be connected to the Kubernetes cluster running the Grafana instance.
`gcloud container clusters get-credentials k8s-workshop-cluster --region europe-north1 --project robocon2024-workshop` This command will set your kube config.

Execute the tests via `robot example-grafana-login.robot`

