# Usage

Prerequisites: helm, kubectl, minikube installed.

## Setup repositories

Create an app repository using [scola84/app-template](https://github.com/scola84/app-template).

Create a job repository using [scola84/job-template](https://github.com/scola84/job-template).

Create a k8s repository using [scola84/k8s-template](https://github.com/scola84/k8s-template).

Clone all three repositories to e.g. `$HOME/dev`.

Run `npm run init` in `$HOME/dev/app` and `$HOME/dev/job`.

Run `npm install` in `$HOME/dev/app` and `$HOME/dev/job`.

## Setup minikube

Start minikube with `minikube start --mount --mount-string="$HOME/dev:/usr/src"`.

Switch to the right kubectl context with `kubectl config use-context minikube`.

## Apply

Apply the k8s config for redis and postgres with `kubectl apply -f $HOME/dev/k8s/local/postgres.yaml -f $HOME/dev/k8s/local/redis.yaml`.

Make sure redis and postgres are running with `kubectl get pods`.

Apply the k8s config for app and job with `kubectl apply -f $HOME/dev/k8s/local/app.yaml -f $HOME/dev/k8s/local/job.yaml -f $HOME/dev/k8s/local/secrets.yaml`.

Install cert-manager with `helm upgrade cert-manager cert-manager --create-namespace --install --namespace cert-manager --repo https://charts.jetstack.io --set installCRDs=true`.

Create a namespace for ingress-nginx with `kubectl create namespace ingress-nginx`.

Enable ingress-nginx with `minikube addons enable ingress`

Retrieve the IP of the cluster with `minikube ip`.

Add `{IP} my.app.local` to `/etc/hosts`.

Apply the k8s config for ingress with `kubectl apply -f $HOME/dev/k8s/local/ingress.yaml`.

Create the database with `kubectl exec deployments/app npm run trun`.

Seed the database with `kubectl exec deployments/app npm run seed`.

Go to `https://my.app.local`.

Optionally, expose app to the local network with `kubectl port-forward services/app 3000:80 --address 0.0.0.0`.

Optionally, expose postgres to the local network with `kubectl port-forward services/postgres 54321:5432 --address 0.0.0.0`.
