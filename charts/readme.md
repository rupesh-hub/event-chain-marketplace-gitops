```bash
kubectl port-forward svc/product-service -n ecm 8181:8181 --address=0.0.0.0 &
kubectl port-forward svc/gateway-server -n ecm 8072:8072 --address=0.0.0.0 &
kubectl port-forward svc/frontend-svc -n ecm 4200:8080 --address=0.0.0.0 &

```