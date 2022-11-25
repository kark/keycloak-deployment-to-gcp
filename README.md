# `Keycloak` deployment to GCP

[Official docs](https://www.keycloak.org/getting-started/getting-started-kube) provide a way to deploy `Keycloak` to local `minikube` cluster. This however doesn't allow to test integration with 3rd-party services, as the URLs are not publicly resolvable. This is the MVP version of the `Keycloak` deployment ready for testing any integrations.

Prerequisite:

- gcp account
- `kubectl` installed
- own domain (can be a free one obtained from https://www.freenom.com/)

Once deployed, it can be accessed via `https://<your-domain>/` (login:`admin`, password: `admin`)

1. Create environment variables

```bash
export PROJECT=your-project  # Your Google Cloud project ID.
export REGION=europe-west1-b   # Your Google Cloud region.
export CLUSTER=keycloak
export EMAIL=your@email.com
export DOMAIN_NAME=your-domain.com
```

1. Create cluster

```bash
gcloud container clusters create $CLUSTER --zone $REGION --preemptible --num-nodes=1
```

2. Create `Keycloak` service and deployment

```bash
kubectl apply -f keycloak.yaml
```

3. Create a static external IP address

```bash
gcloud compute addresses create ip-keycloak --global
```

You should see the new IP address listed:

```bash
gcloud compute addresses list
```

4. Point your domain to the static external IP

In your domain DNS records create a new `A` record (without a name) pointing to the static external IP and wait for the changes to propagate

5. Create secret

```bash
kubectl apply -f secret.yaml
```

6. Install cert-manager and create an issuer for Let's Encrypt staging

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.2/cert-manager.yaml
kubectl apply -f issuer-letsencrypt-staging.yaml
```

7. Create an Ingress

Edit `keycloak-ingress.yaml`

```diff
 spec:
   tls:
     hosts:
-      - your-domain.com
+      - your-real-domain.com
```

Then:

```bash
kubectl apply -f keycloak-ingress.yaml
```

Check progress with:

```bash
kubectl describe ingress keycloak
```

You may verify if all is set up correctly with:

```bash
curl -v --insecure https://$DOMAIN_NAME
```

7. Cleanup

```bash
gcloud container clusters delete $CLUSTER --zone $REGION
gcloud compute addresses delete ip-keycloak --global
gcloud certificate-manager certificates delete keycloak
```
