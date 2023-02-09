# Usage

Prerequisites: doctl, gh, helm and kubectl installed.

Change the host in both `rules` and `tls` in the Ingress in `$HOME/dev/k8s/remote/ingres.yaml`.

Change the email address in `acme` in the ClusterIssuer in `$HOME/dev/k8s/remote/ingress.yaml`.

Change the image location in `containers` in the Deployment in both `$HOME/dev/k8s/remote/app.yaml` and `$HOME/dev/k8s/remote/job.yaml`.

## Create access tokens

Create two personal access tokens (PAT) at Github. The first should have read and write access to repositories and packages and the second should only have read access to packages.

Add the first PAT as a repository secret with the name `ACCESS_TOKEN` to the app and job repositories at Github. This secret is used in the Github build workflow.

Save the second PAT to `/tmp/secrets/ghcr-secret`. This secret is used in the Kubernetes cluster to pull private images from Github.

## Push, release and build

Push changes to `origin/main` in the app and job repositories.

Retrieve the new image versions of app and job with `gh workflow view build` in both repositories.

Change the image versions in `$HOME/dev/k8s/remote/app.yaml` and `$HOME/dev/k8s/remote/job.yaml`.

## Create resources

### DigitalOcean

Create a Kubernetes cluster with `doctl kubernetes cluster create ... --auto-upgrade --region ... --set-current-context --update-kubeconfig`.

Create a PostgreSQL database with `doctl databases create ... --engine pg --region ...`.

Save the (private) PostgreSQL URI to `/tmp/secrets/postgres-uri`.

Save the PostgreSQL CA to `/tmp/secrets/postgres-ca`.

Restrict access to the PostgreSQL database to the Kubernetes cluster.

Create a Redis database with `doctl databases create ... --engine redis --region ...`.

Save the (private) Redis URI to `/tmp/secrets/redis-uri`.

Restrict access to the Redis database to the Kubernetes cluster.

## Create secrets

Save a session secret to `/tmp/secrets/session-secret`.

Create the ghcr-secret with `kubectl create secret docker-registry ghcr-secret --docker-server=ghcr.io --docker-username=... --docker-password=$(cat /tmp/secrets/ghcr-secret) --docker-email=...`. The `--docker-username` and `--docker-email` should reflect the user for which the second PAT was created.

Create the postgres-ca secret with `kubectl create secret generic postgres-ca --from-file /tmp/secrets/postgres-ca`.

Create the postgres-uri secret with `kubectl create secret generic postgres-uri --from-file /tmp/secrets/postgres-uri`.

Create the redis-uri secret with `kubectl create secret generic redis-uri --from-file /tmp/secrets/redis-uri`.

Create the session secret with `kubectl create secret generic session-secret --from-file /tmp/secrets/session-secret`.

Remove the secrets from disk with `rm /tmp/secrets/*`.

## Apply

Apply the k8s config for app and job with `kubectl apply -f $HOME/dev/k8s/remote/app.yaml -f $HOME/dev/k8s/remote/job.yaml`.

Install cert-manager with `helm upgrade cert-manager cert-manager --create-namespace --install --namespace cert-manager --repo https://charts.jetstack.io --set installCRDs=true`.

Install ingress with `helm upgrade ingress-nginx ingress-nginx --create-namespace --install --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx`

Acquire the IP of the load balancer with `kubectl get services --namespace ingress-nginx ingress-nginx-controller`.

Add the IP to the DNS.

Apply the k8s config ingress with `kubectl apply -f $HOME/dev/k8s/remote/ingress.yaml`.

Check the status of the certificate with `kubectl describe certificate letsencrypt`.

Create the database with `kubectl exec deployments/app npm run trun`.

Seed the database with `kubectl exec deployments/app npm run seed`.

Go to the app.
