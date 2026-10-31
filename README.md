# k3s-manifests

Kubernetes manifests for a k3s cluster: Deployment, Service, Ingress (Traefik), PersistentVolumeClaim.

## Apply

```bash
kubectl apply -f pvc.yaml -f deployment.yaml -f service.yaml -f ingress.yaml
kubectl get pods,svc,ingress,pvc
```

## Notes

- `local-path` is the default k3s storage class (Local Path Provisioner).
- Ingress is served by the bundled Traefik controller.
- Scale: `kubectl scale deployment web --replicas=4`.
