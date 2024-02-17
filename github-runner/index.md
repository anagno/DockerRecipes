# Self-hosted Github runner

For it will need to first create a github Personal Acces Token. For that follow the instructions 
[here](https://github.com/actions/actions-runner-controller/blob/master/docs/authenticating-to-the-github-api.md#deploying-using-pat-authentication).

To store our files, we will use nextcloud

``` bash
kubectl create namespace github-runner

kubectl create secret generic controller-manager \
      -n github-runner \
      --from-literal=github_token=YOUR_ACCESS_TOKEN

helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller
helm repo update
helm install --namespace github-runner github-controller actions-runner-controller/actions-runner-controller -f values.yaml --version 0.23.7
kubectl apply -f runner.yaml


```

## Resources:

* https://medium.com/mossfinance/github-self-hosted-runners-on-kubernetes-with-actions-runner-controller-41e30c4cb76e
* https://github.com/actions/actions-runner-controller?tab=readme-ov-file#legacy-documentation
* https://www.velotio.com/engineering-blog/how-to-deploy-github-actions-self-hosted-runners-on-kubernetes
* https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller/quickstart-for-actions-runner-controller
