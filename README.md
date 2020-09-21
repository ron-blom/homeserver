# Homeserver based on k3os with multus-cni

### VLAN's:
```
- 110: 192.168.110.0/24 Internal
- 111: 192.168.111.0/24 Guest (Only WiFi)
- 112: 192.168.112.0/24 Management
```

### POD ip's and URL's:
```
192.168.112.3   omada-controller-pod    https://192.168.112.3:8043/login
192.168.112.4   freepbx-pod             http://192.168.112.4/
192.168.110.5   squeezebox-pod          http://192.168.110.5:9000/
192.168.110.6   dsmr-reader-pod         http://192.168.110.6/
192.168.110.7   diyhue-pod              
192.168.110.8   kerberos-pod            http://192.168.110.8
192.168.110.10  cups-pod                https://192.168.110.10:631
```

Download latest k3os-amd64.iso from https://github.com/rancher/k3os/releases
Create bootable usb stick with k3os.iso

Boot from memory stick and install k3os.
Use cloud init with url: https://raw.githubusercontent.com/ron-blom/homeserver/master/init-config.yaml

Login to new server and create config.yaml:
`sudo  vi /var/lib/rancher/k3os/config.yaml`
paste content of config.yaml

Copy kube config to your desktop and modify ip address:
```
scp rancher@192.168.110.2:/etc/rancher/k3s/k3s.yaml ~/.kube/config
vi ~/.kube/config
```

### Multus-CNI
modified multus-daemonset.yml from project: clone https://github.com/intel/multus-cni.git

`kubectl apply -f multus-daemonset.yml`

kopieer multus binary to /var/lib/rancher/k3s/data/4b08ff55a6cec1960f6f97519da0ac5351986d629107a1ef4a1e216ff5579bf0/bin (from /opt/cni/bin/)

build: https://github.com/containernetworking/plugins.git
with linux script.

Copy binary’s to: /opt/cni/bin

# add networking config to k8s
```
kubectl apply -f macvlan.yaml
```

# Install ingress
```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update
helm install nginx-ingress stable/nginx-ingress --set controller.service.LoadBalancerIP=192.168.110.2 --set controller.service.externalIPs={192.168.110.2}
```