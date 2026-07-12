# kind / ingress local access

## Problem

The cluster is using an `Ingress` resource, but local access to `http://my-app.local` failed because the ingress controller is not listening on host port `80`.

## Immediate workaround

If you want to test quickly without recreating the cluster:

```bash
sudo kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 80:80
```

Then in another terminal:

```bash
echo "127.0.0.1 my-app.local" | sudo tee -a /etc/hosts
curl http://my-app.local
```

> You need `sudo` for binding local port `80`.

## Better local setup with kind

1. Recreate the cluster using `kind-config.yaml`:

```bash
kind delete cluster
kind create cluster --config kind-config.yaml
```

2. Install or reinstall the ingress controller:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

3. Apply your app manifests:

```bash
kubectl apply -f service.yaml
kubectl apply -f deployment.yaml
kubectl apply -f ingress.yaml
```

4. Confirm the controller is reachable:

```bash
curl -I http://my-app.local
```

## Notes

- `service.yaml` should remain `ClusterIP` when using an ingress.
- `ingress.yaml` routes `my-app.local` to `my-app-service`.
- If port `80` is already used by another process, use the port-forward workaround with a different local port.
