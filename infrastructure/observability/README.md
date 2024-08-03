# Observability

## Grafana Ingress

> http://grafana.cursedcompass.ddns.net:31375

Find the bound nginx HTTP port. In this case it is 31375.

```bash
$ kubectl get service -n ingress-nginx

NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.152.183.183   <none>        80:31375/TCP,443:32160/TCP   156m
ingress-nginx-controller-admission   ClusterIP   10.152.183.129   <none>        443/TCP                      156m
```

Setup network to port forward port 31375.

Configure the Grafana values to create an nginx ingress for path `/grafana`. You'll also need to configure `grafana.ini` to set the root URL, else redirects such as `login` will go to `/`, resulting in a 404. 
