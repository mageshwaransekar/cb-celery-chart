# Helm Chart to deploy Redis and DJango-Celery

## Generate Helm chart

This chart is initially generated using:
```
helm create cargobase-celery-chart
```

## NGINX Ingress
Before deploying the main chart, we will be first creating the namespace. Here, we are using `backend` namespace:

```
kubectl create ns backend
```

And then, we will be deploying nginx ingress controller:
```
cd misc/
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm upgrade --install --namespace backend --version 4.7.0 nginx-ingress ingress-nginx/ingress-nginx -f nginx-values.yaml
```

## Deploying django-celery chart
### Changes to the variables
Go back to the root folder
```
cd ..
```
Before deploying the helm chart, there are 2 variables that need to be replaced in `values.yaml`:

```
djangoCelery:
  image: <django-docker-image-here>
```
Replace the value with the ECR repo created using terraform. For example: `public.ecr.aws/sndf67qtdv/django-celery:latest`

```
django:
  ingress:
    host: <elb-dns-name-here>
```
Replace the value with elb created by the nginx ingress controller. This is the DNS name of ELB found under `EC2 > Loadbalancer`

### Helm Install
Now we can deploy the chart:
```
helm upgrade --install -f values.yaml backend .
```

## Liveness Probe
We have enabled liveness probe for django-celery in order to check if the process has slowed down or become non-responsive. In that case, the pod will be restarted and the process will be back to normal.

To simulate this, we can access the URL of <app-url>/slow and open multiple tab or curl. And monitor whether the pods are restarted. In our case <app-URL> is the DNS name of the ELB.