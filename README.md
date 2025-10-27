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
192.168.114.4   freepbx-pod             http://192.168.114.4/
192.168.110.5   squeezebox-pod          http://192.168.110.5:9000/
192.168.110.6   dsmr-reader-pod         http://192.168.110.6/
192.168.110.7   diyhue-pod              
192.168.110.8   kerberos-pod            http://192.168.110.8
192.168.112.9   webserver-pod           http://homeserver.local or http://192.168.112.9
192.168.110.10  cups-pod                https://192.168.110.10:631
192.168.114.10  tftp-server             tftp://192.168.114.10:69
192.168.110.11  pihole-pod              http://192.168.110.11
192.168.110.12  squid                   http://192.168.110.12:3128
```

### Alma9 add vlan
[root@k3os ~]# nmcli con mod enp2s0f0 ipv4.addresses 192.168.110.2/24
[root@k3os ~]# nmcli con mod enp2s0f0 ipv4.gateway 192.168.110.1
[root@k3os ~]# nmcli con mod enp2s0f0 ipv4.dns "8.8.8.8"
[root@k3os ~]# nmcli con mod enp2s0f0 ipv4.dns "192.168.110.1"
[root@k3os ~]# nmcli con mod enp2s0f0 ipv4.method manual

[root@k3os ~]# nmcli connection add type vlan con-name vlan112 ifname vlan112 vlan.parent enp2s0f0 vlan.id 112
[root@k3os ~]# nmcli con mod vlan112 ipv4.addresses 192.168.112.2/24
[root@k3os ~]# nmcli con mod vlan112 ipv4.method manual

[root@k3os ~]# nmcli connection add type vlan con-name vlan114 ifname vlan114 vlan.parent enp2s0f0 vlan.id 114
[root@k3os ~]# nmcli con mod vlan114 ipv4.addresses 192.168.114.2/24
[root@k3os ~]# nmcli con mod vlan114 ipv4.method manual


### Install k3s
[root@k3os ~]# systemctl disable firewalld --now

[root@k3os ~]# curl -sfL https://get.k3s.io | sh -s - --disable=traefik

Copy /etc/rancher/k3s/k3s.yaml to local ~/.kube/config

### Multus-CNI
rblom@Rons-MacBook-Air git % git clone git@github.com:k8snetworkplumbingwg/multus-cni.git
rblom@Rons-MacBook-Air git % cd multus-cni

Change deployments/multus-daemonset-thick.yml

      "kubeconfig": "/var/lib/rancher/k3s/agent/etc/cni/net.d/multus.d/multus.kubeconfig"

      volumes:
        - name: cni
          hostPath:
            path: /var/lib/rancher/k3s/agent/etc/cni/net.d
        - name: cnibin
          hostPath:
            path: /var/lib/rancher/k3s/data/current/bin

rblom@Rons-MacBook-Air multus-cni % cat ./deployments/multus-daemonset.yml | kubectl apply -f -

### 
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.26/deploy/local-path-storage.yaml

### Ingress
rblom@Rons-MacBook-Air homeserver % helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
rblom@Rons-MacBook-Air homeserver % helm install ingress-nginx ingress-nginx/ingress-nginx






## OLD

Download latest k3os-amd64.iso from https://github.com/rancher/k3os/releases
Create bootable usb stick with k3os.iso

Boot from memory stick and install k3os.
Use cloud init with url: https://raw.githubusercontent.com/ron-blom/homeserver/master/init-config.yaml

Login to new server and create config.yaml:
```
sudo  vi /var/lib/rancher/k3os/config.yaml
```
paste content of config.yaml
Copy kube config to your desktop and modify ip address:
```
scp rancher@192.168.110.2:/etc/rancher/k3s/k3s.yaml ~/.kube/config
vi ~/.kube/config
```

### Multus-CNI
modified multus-daemonset.yml from project: clone https://github.com/intel/multus-cni.git

`kubectl apply -f multus-daemonset.yml`

copy multus binary to /var/lib/rancher/k3s/data/4b08ff55a6cec1960f6f97519da0ac5351986d629107a1ef4a1e216ff5579bf0/bin (from /opt/cni/bin/)

build: https://github.com/containernetworking/plugins.git
with linux script.

Copy binary’s to: /opt/cni/bin

add networking config to k8s
```
kubectl apply -f macvlan.yaml
```

# Install ingress
```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update
helm install nginx-ingress stable/nginx-ingress --set controller.service.LoadBalancerIP=192.168.110.2 --set controller.service.externalIPs={192.168.110.2}
helm secrets upgrade --install homeserver-chart ./homeserver-chart -f homeserver-chart/secrets.yaml
```