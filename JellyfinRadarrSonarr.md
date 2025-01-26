# jellyfin + radarr/sonarr server

1. Create the media Namespace

```sh
kubectl create namespace media
```

2. Install/Configure Components in the `media` Namespace

2.1. Storage Provisioner

```sh
kubectl create namespace local-path-storage
```

```sh
kubectl apply -n local-path-storage \
  -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

```sh
kubectl get pods -n local-path-storage
kubectl get sc
```

```sh
kubectl patch sc local-path \
  -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
