# Apply the YAML file using this command

```sh
kubectl apply -f k8s_media_stack.yaml


namespace/multimedia unchanged
persistentvolume/plex-pv unchanged
persistentvolumeclaim/plex-pvc unchanged
deployment.apps/plex unchanged
service/plex-service unchanged
```

### Verify the Deployment

1. Namespace:

```sh
kubectl get namespace
```

2. Persistent Volume and Claim:

```sh
kubectl get pv,pvc -n multimedia
```

3. Deployment and Pods:

```sh
kubectl get pods -n multimedia
```

4. Service:

```sh
kubectl get svc -n multimedia
```

# Access the Plex Application

```sh
hostname -I
```
