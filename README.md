# Home Server Flux Setup

## MetalLB

Allows you to create load balancers in your Kubernetes cluster.

We need to reserve an IP address range for MetalLB to use.

The router's current DHCP address pool is 192.168.0.2 - 192.168.0.253. We will shrink it to 192.168.0.2 - 192.168.0.239 to give us the following available address pool for MetalLB: 192.168.0.240 - 192.168.0.250.  

## Trivy

Vulnerability scanner.

```bash
kubectl get vulnerabilityreports --all-namespaces -o wide 
kubectl get configauditreports --all-namespaces -o wide 
```

## Ingress

http://192.168.0.240

## Observability

http://192.168.0.242

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

### Provisioned dashboards/alerts/contact-points

Sidecars are enabled in the Grafana chart values. The sidecar checks for Kubernetes resources labelled with `grafana_dashboard` and `grafana_alert`. These resources are YAML files which can be obtained by exporting from the Grafana UI.

The alerts are added to Kubernetes as a config map.

The contact-points are added to Kubernetes as a secret because they can contain sensitive values. In this case it contains a personal email address. This is the process for creating the sealed secret.

1. Create the `contact-points.yaml`:

    ```yaml
    apiVersion: 1
    contactPoints:
      - orgId: 1
        name: grafana-default-email
        receivers:
          - uid: a
            type: email
            settings:
              addresses: changeme
              singleEmail: false
            disableResolveMessage: false
    ```

2. Base64 encode the contents of the file:

    ```bash
    cat contact-points.yaml | base64 > contact-points.yaml.enc
    ```

3. Create the YAML file `contact-points-secret.yaml` and add the base64 encoded contents to the secret:

   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: grafana-contact-points
     namespace: observability
     labels:
       grafana_alert: "1"
   data:
     contact-points.yaml: |
       line1
       line2
   ```

4. Seal the secret:

   ```bash
   kubeseal --format=yaml --cert=pub-sealed-secrets.pem \
     < contact-points-secret.yaml > contact-points-sealed.yaml
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
