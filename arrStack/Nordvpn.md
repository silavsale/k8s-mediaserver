1. Create the Secrets
   Create the secret for the OpenVPN configuration file (rename your file as client.ovpn):

bash
Copy
kubectl delete secret vpn-config -n media 2>/dev/null
kubectl create secret generic vpn-config \
 --from-file=client.ovpn=/home/sergo/Downloads/ca1638.nordvpn.com.udp.ovpn \
 -n media
Create the secret for authentication. First, create a local file named auth.txt with two lines:

nginx
Copy
your_nordvpn_username
your_nordvpn_password
Then run:

bash
Copy
kubectl delete secret vpn-auth -n media 2>/dev/null
kubectl create secret generic vpn-auth --from-file=auth.txt=./auth.txt -n media 2. Create the ConfigMap for the Route Override Script
Create a file named route-override.sh with the following content:

sh
Copy
#!/bin/sh
VPN_GATEWAY=$(route -n | awk 'NR==3 {print $2}')
ip route del 0.0.0.0/1 via $VPN_GATEWAY
echo "Route Updated"
Then create the ConfigMap:

bash
Copy
kubectl delete configmap route-script -n media 2>/dev/null
kubectl create configmap route-script --from-file=route-override.sh=./route-override.sh -n media 3. Deployment and Service YAML
Save the following YAML to a file (for example, openvpn-deployment.yaml):

yaml
Copy
apiVersion: apps/v1
kind: Deployment
metadata:
name: openvpn-client
namespace: media
spec:
replicas: 1
selector:
matchLabels:
app: openvpn-client
vpn: nordvpn
template:
metadata:
labels:
app: openvpn-client
vpn: nordvpn
spec:
volumes: - name: vpn-config
secret:
secretName: vpn-config
items: - key: client.ovpn
path: client.ovpn - name: vpn-auth
secret:
secretName: vpn-auth
items: - key: auth.txt
path: auth.txt - name: route-script
configMap:
name: route-script
items: - key: route-override.sh
path: route-override.sh - name: tmp
emptyDir: {}
initContainers: - name: vpn-route-init
image: busybox
command: ["/bin/sh", "-c", "mkdir -p /tmp/route && cp /vpn/route-override.sh /tmp/route/route-override.sh && chmod +x /tmp/route/route-override.sh"]
volumeMounts: - name: tmp
mountPath: /tmp/route - name: route-script
mountPath: /vpn/route-override.sh
subPath: route-override.sh
containers: - name: vpn
image: dperson/openvpn-client
command: ["/bin/sh", "-c"]
args: ["openvpn --config 'vpn/client.ovpn' --auth-user-pass 'vpn/auth.txt' --script-security 3 --route-up /tmp/route/route-override.sh; sleep infinity"]
tty: true
stdin: true
securityContext:
privileged: true
capabilities:
add: - NET_ADMIN
volumeMounts: - name: vpn-config
mountPath: /vpn/client.ovpn
subPath: client.ovpn - name: vpn-auth
mountPath: /vpn/auth.txt
subPath: auth.txt - name: tmp
mountPath: /tmp/route
env: - name: TZ
value: "UTC"
dnsConfig:
nameservers: - 8.8.8.8 - 8.8.4.4

---

apiVersion: v1
kind: Service
metadata:
name: openvpn-client
namespace: media
spec:
selector:
app: openvpn-client
ports: - name: dummy
protocol: TCP
port: 9999
targetPort: 9999
type: ClusterIP 4. Deploy the Resources
Apply the Deployment and Service:

bash
Copy
kubectl apply -f openvpn-deployment.yaml 5. Verify the Setup
Check Pod Status:

bash
Copy
kubectl get pods -n media
Exec into the VPN Container to Verify Mounts:

Replace <pod-name> with the name of your running pod:

bash
Copy
kubectl exec -it <pod-name> -n media -- ls -lah /vpn
kubectl exec -it <pod-name> -n media -- ls -lah /tmp/route
You should see client.ovpn and auth.txt in /vpn and route-override.sh in /tmp/route.

Test the VPN Connection:

bash
Copy
kubectl exec -it <pod-name> -n media -- curl ifconfig.me
