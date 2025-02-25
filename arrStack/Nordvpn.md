# Step 1: Create NordVPN Pod

We'll deploy NordVPN using OpenVPN inside a Kubernetes pod.

1️. Create a Secret for NordVPN Credentials

```bash
kubectl create secret generic nordvpn-credentials \
  --from-literal=username='your_nordvpn_username' \
  --from-literal=password='your_nordvpn_password' -n media
```

2. Download NordVPN P2P OpenVPN Config File

Find a P2P server at NordVPN’s Server List.

Download a UDP P2P server config (recommended for torrents):

```bash
wget -O ca1638.nordvpn.ovpn https://downloads.nordcdn.com/configs/files/ovpn_udp/ca1638.nordvpn.com.udp.ovpn
```

Then, create a ConfigMap with this .ovpn file:

```bash
kubectl create configmap nordvpn-config --from-file=nordvpn.ovpn -n media
```

3️. Deploy the NordVPN Pod (nordvpn.yaml)

Create a deployment file for NordVPN:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nordvpn
  namespace: media
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nordvpn
  template:
    metadata:
      labels:
        app: nordvpn
    spec:
      containers:
        - name: nordvpn
          image: dperson/openvpn-client
          securityContext:
            capabilities:
              add:
                - NET_ADMIN
          env:
            - name: OPENVPN_USER
              valueFrom:
                secretKeyRef:
                  name: nordvpn-credentials
                  key: username
            - name: OPENVPN_PASS
              valueFrom:
                secretKeyRef:
                  name: nordvpn-credentials
                  key: password
          args:
            - '-f'
            - '--config'
            - '/vpn-config/nordvpn.ovpn'
          volumeMounts:
            - name: vpn-config
              mountPath: '/vpn-config'
      volumes:
        - name: vpn-config
          configMap:
            name: nordvpn-config
---
apiVersion: v1
kind: Service
metadata:
  name: nordvpn
  namespace: media
spec:
  selector:
    app: nordvpn
  type: ClusterIP
```

4. Apply the NordVPN Deployment

```bash
 kubectl apply -f nordvpn.yaml
```

✅ Step 2: Test If NordVPN Is Working
Now that the NordVPN pod is running, let's verify the connection.

1️⃣ Check Logs to Confirm VPN Connection
bash
Copy
Edit
kubectl logs -f deployment/nordvpn -n media
✅ You should see logs showing a successful connection to NordVPN.

2️⃣ Run an IP Check From the NordVPN Pod
Exec into the NordVPN container and check its public IP:

bash
Copy
Edit
kubectl exec -it deployment/nordvpn -n media -- curl ifconfig.me
✅ The output should show a NordVPN P2P server IP, not your real IP.

3️⃣ Verify Connection Using a Leak Test
Inside the pod:

bash
Copy
Edit
kubectl exec -it deployment/nordvpn -n media -- wget -qO- https://ipleak.net

```

```
