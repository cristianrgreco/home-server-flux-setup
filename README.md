# Home Server Flux Setup

## MetalLB

> Allows you to create load balancers in your Kubernetes cluster.

We need to reserve an IP address range for MetalLB to use.

The router's current DHCP address pool is 192.168.0.2 - 192.168.0.253. We will shrink it to 192.168.0.2 - 192.168.0.239 to give us the following available address pool for MetalLB: 192.168.0.240 - 192.168.0.250.  

> [!NOTE]  
> We should install MetalLB via Helm + Flux in the future: https://metallb.universe.tf/installation/


```bash
microk8s enable metallb:192.168.0.240-192.168.0.251
```

### Ingress

Find the bound nginx HTTP port. In this case it is 31375.

```bash
$ kubectl get service -n ingress-nginx

NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.152.183.183   <none>        80:31375/TCP,443:32160/TCP   156m
ingress-nginx-controller-admission   ClusterIP   10.152.183.129   <none>        443/TCP                      156m
```

Setup router to port forward port 31375.

Configure the Grafana values to create an nginx ingress for path `/grafana`. You'll also need to configure `grafana.ini` to set the root URL, else redirects such as `login` will go to `/`, resulting in a 404.

## Observability

http://192.168.0.244

### Admin user

Get grafana admin credentials:

```bash
kubectl get secrets/grafana-adminuser-creds -n observability -o json \
  | jq -r '.data.adminUser' | base64 -d

kubectl get secrets/grafana-adminuser-creds -n observability -o json \
  | jq -r '.data.adminPassword' | base64 -d
```

For posterity, this is how they were created:

```bash
kubectl -n observability create secret generic grafana-adminuser-creds \
  --from-literal=adminUser='changeme' \
  --from-literal=adminPassword='changeme' \
  --dry-run=client -o yaml \
  > auth.yaml
  
kubeseal --format=yaml --cert=pub-sealed-secrets.pem \
  < auth.yaml > auth-sealed.yaml
  
rm auth.yaml
```

## Pihole

http://192.168.0.241/admin

### Admin user

Get pihole admin user password:

```bash
kubectl get secrets/pihole-adminuser -n pihole -o json \
  | jq -r '.data.password' | base64 -d
```

## Useful commands

```bash
microk8s status
flux get kustomizations --watch
flux events --watch
flux get helmreleases --all-namespaces
flux reconcile hr -n minecraft minecraft
flux suspend helmrelease -n minecraft minecraft
flux resume helmrelease -n minecraft minecraft
```

# TODO

1. MetalLB to ansible
2. Hardcode LB IPs in prod values so re-deployment wouldn't change
3. Create IP address reservation for the pi in router settings