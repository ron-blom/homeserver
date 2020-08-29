# Homeserver based on k3os with multus-cni

VLAN's:
- 110: 192.168.110.0/24 Internal
- 111: 192.168.111.0/24 Guest (Only WiFi)
- 112: 192.168.112.0/24 Management

POD ip's and URL's:
192.168.112.3   omada-controller-pod    https://192.168.112.3:8043/login
192.168.112.4   freepbx-pod             http://192.168.112.4/
192.168.110.5   squeezebox-pod          http://192.168.110.5:9000/
192.168.110.6   dsmr-reader-pod         http://192.168.110.6/
192.168.110.7   diyhue-pod              


git clone https://github.com/intel/multus-cni.git
cd ../multus-cni

Paden aanpassen in multus-daemonset.yml 
/var/lib/rancher/k3s/agent/etc/… i.p.v. /etc/…

kubectl apply -f images/multus-daemonset.yml

kopieer multus binary naar /var/lib/rancher/k3s/data/ 4b08ff55a6cec1960f6f97519da0ac5351986d629107a1ef4a1e216ff5579bf0/bin (vanaf /opt/cni/bin/)

Bouw: https://github.com/containernetworking/plugins.git
dmv linux script.

Copieer binary’s naar: /opt/cni/bin



