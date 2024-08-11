# Home Server Flux Setup

## MetalLB

- DHCP address range
- Init flags, etc

## Observability

http://cursedcompass.ddns.net:31375/grafana

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

## Pihole

http://cursedcompass.ddns.net:30001/admin

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

1. Grafana to use LB
2. Pihole to use LB
3. Configure router to use pihole
4. MetalLB to ansible
   1. Consider doing it all via flux
5. Do not port forward and expose internal services
6. Update docs
7. Hardcoded LB IPs in prod values so re-deployment wouldn't change
