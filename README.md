# To-do-app
# AI Planet Devops Assignment

## Objective
The project involves setting up a GitOps pipeline to automate the deployment and management of a simple web application. utilizing Argo CD for continuous deployment and Argo Rollouts for advanced deployment strategies within a Kubernetes environment.

## Prerequisites
- Python
- Flask
- Docker
- Git and Github
- Github Action
- Kubernetes
- Argo CD and Argo Rollouts

## Workflow in the CI/CD pipeline
- Create a Flask application
- Dockerise the application
- Create 2 GitHub repositories one for storing our application code and the other for storing our Manifest files for Kubernetes.
- Set up Argo CD to monitor your repository and automatically deploy changes to your Kubernetes cluster(Minikube) using Canary release with Argo Rollout.
- Use GitHub actions as a CI tool to build, test, and push the docker image to Dockerhub and update the Kubernetes Manifest files.


## Project structure
**app.py** - Flask application which is a simple to-Do List web application.

**requirements.txt** - Contains dependencies for the project

**Dockerfile** - Contains commands to build and run the docker image

**.github/workflows/main.yaml** - Contains the pipeline script which will help in building, testing, and deploying the application 

_Note_: As it is recommended to have the Manifest files which are used to create Kubernetes clusters in different repositories, I have created them here [Github Link](https://github.com/nikhilp047/gitops-infra)

**dev/deployment.yaml** - Kubernetes deployment file for the application

**dev/service.yaml** - Kubernetes service file for the application

**application.yaml** - file to create an ARGO CD application in Kubernetes

## Web app preview
<img width="815" alt="image" src="https://github.com/nikhilp047/To-do-app/assets/43903557/db79eb2e-9ffe-46b6-9986-d5f84109fcaa">


Install **Docker desktop** by following the steps provided in the below link:
(https://docs.docker.com/desktop/install/windows-install/)

## MiniKube 

Since a single cluster was enough for a demo purpose I used Minikube which I was running on my local machine. You can refer to the below link to install Minikube on a local machine (https://minikube.sigs.k8s.io/docs/start/) 

Note: Need to make sure that you have installed Docker Desktop on your machine and that it's running while using Minikube

Install **Minikube** in local machine using the below link:
(https://minikube.sigs.k8s.io/docs/start/)

Start the cluster by using
```
minikube start
```

## ArgoCD
Next we will install **argocd** to run in our cluster
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
## ArgoCD Rollot
We can install Argo Rollouts by running:
```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

```


## Access The Argo CD Dashboard
By default, the Argo CD API server is not exposed with an external IP. To access the API server,
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Now you can access it at (https://localhost:8080)


## Creating the app with Argo CD
- We now need to create an app for the deployment and service:
- Either you can manually create a ArgoCD application or you can use an argocd application manifest yaml file.
   - I have used an application manifest file you can find it in [this repository](https://github.com/nikhilp047/gitops-infra)
   - Run the below commandd to create an app for the deployment
     ```
      kubectl apply -f application.yaml
     ```

## Argo CD Dashboard
![image](https://github.com/nikhilp047/To-do-app/assets/43903557/de209eef-7ddd-4639-831f-affe44105d0e)
- We can observe our Argo CD app is healthy and is synced to our latest commits in the repository (https://github.com/nikhilp047/gitops-infra).
- The Kubernetes application was successfully deployed. According to the configuration in deployment.yaml, 4 replicas were asked and that is why 4 pods are visible on the dashboards.

## Canary Rollout Strategy Release
The first rollout is the intial rollout so it will create pods normally. The subsequent rollouts will be following the Canary release strategy which we had defined in deployment.yaml
```
kind: Rollout
metadata:
  name: myapp-rollouts
spec:
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: {}
        - setWeight: 40
        - pause: {duration: 10}
        - setWeight: 60
        - pause: {duration: 10}
        - setWeight: 80
        - pause: {duration: 10}
```

The strategy will follow the below steps:

- We redirect 20% of the traffic to our new version The remaining 80% remains with the previous release
  - Before it moves to the next step it will wait for human Intervention we will have to promote our deployment to the next step
- Then 40% of requests to our new version
- Then 60% after waiting for 10 seconds
- Then 80% after 10 seconds
- At the end, 100% of the traffic is directed towards the new version of pods


Commit and pushing to the [repo](https://github.com/nikhilp047/gitops-infra) will automatically update the kubernetes deployments with the help of Argo CD's synchronization. 

After the duration of the canary release, all previous release pods will be terminated and the result is a  successful release of new version.
![image](https://github.com/nikhilp047/To-do-app/assets/43903557/380af78f-c9d0-49f9-a579-19f760676b9d)
![image](https://github.com/nikhilp047/To-do-app/assets/43903557/3a6e5d73-a129-4d29-9321-18b3ff617963)

## Github action workflow
The github action is triggered whenever push action is performed on this repository
![image](https://github.com/nikhilp047/To-do-app/assets/43903557/bed5de0b-0616-404f-9df6-87152d8d62f3)


- In the first step of the workflow we test and check the code
- Next, we are building and pushing the Docker image to our Dockerhub
- After the above steps are successfully completed we will update the manifest files used by kubernetes [this repository](https://github.com/nikhilp047/gitops-infra), which will be monitored by ArgoCD.
- Then the ArgoCD will automatically deploy changes to your Kubernetes cluster(Minikube) using Canary release with Argo Rollout.




## Cleaning up
Let's delete the rollouts -
```
kubectl delete rollouts myapp-rollouts -n myapp-rollout
```

Deleting the app from Argo CD -
```
argocd app delete api --cascade
```
Although the above command can be used to delete application, i was facing some issue with the ArgoCD cli so i deleted the application using the ArgoCD UI .

Next stop the terminals that is used for portforwarding and You can delete this instance of minikube with -
```
minikube delete
```


