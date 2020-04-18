# Udagram microservices:

Github URL: https://github.com/socket-var/udagram-microservices

This project:

- Divides the application into smaller services
- Containerizes the application, create the Kubernetes resource, and deploy it to a Kubernetes cluster.
- Implements automatic continuous integration (CI) and continuous delivery (CD) using Travis CI.
- Extends the application with deployments and be able to do rolling-updates and rollbacks

## Instructions:

- Create aws-secret.yaml, env-secret.yaml and store the appropriate secrets.
- Change the config variables in env-configmap.yaml
- Install eksctl by following instructions from https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html
- Follow instructions from the udacity course to deployt to eks using kubectl commands
- Below screenshot shows that the Kubernetes cluster has been successfully created:

![](./screenshots/eks_deployment.png)

- Below is the screenshot obtained by running `% kubectl get all`

![](./screenshots/kubectl_get_all.png)

- Follow the instructions in the udacity course to add a .travis.yaml file in the root of your project
- Below is the screenshot that shows a successful build with Travis CI on push to Github:

![](./screenshots/travis-ci.png)

- The following snippet from .travis.yaml shows the steps and configuration parameters to be additionally set for doing CD using kubectl:

```yaml
before_install:
  # .......redacted docker setup commands for brevity.............
  - curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
  - chmod +x ./kubectl
  - sudo mv ./kubectl /usr/local/bin/kubectl
  - mkdir ${HOME}/.kube
  - cp udacity-c3-deployment/k8s/config ${HOME}/.kube/config
  - kubectl config set clusters.development.certificate-authority-data "$KUBE_CLUSTER_CERTIFICATE"
  - kubectl config set users.developer.client-certificate-data "$KUBE_CLIENT_CERTIFICATE"
  - kubectl config set users.developer.client-key-data "$KUBE_CLIENT_KEY"

install:
    # .....redacted docker compose commands for brevity..........
  - kubectl apply -f udacity-c3-deployment/k8s/backend-feed-service.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/backend-user-service.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/frontend-service.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/reverseproxy-service.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/aws-secret.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/env-configmap.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/env-secret.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/backend-feed-deployment.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/backend-user-deployment.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/frontend-deployment.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/reverseproxy-deployment.yaml
```

- In the before install steps we need to move our kubeconfig file to $HOME path so that kubectl can read our cluster's configuration information from there.

- We apply the yaml files corresponding to our microservices to deploy.

- `kubectl set image` command can be used to initiate a rolling update as follows:

```
% kubectl set image deployments/udacity-restapi-feed udacity-restapi-feed=sakethpericherla/udacity-restapi-feed:v2
```

- Similarly the changes can be rolled back using the undo command as follows:

```
kubectl rollout undo deployments/udacity-restapi-feed
```
