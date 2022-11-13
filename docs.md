# Get machine info
```
lsb_release -a
htop
```

# Test cli tools
```
docker
kind
kubectl
```

# Install kind
```sh
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

# Install kubectl autocompletion
```sh
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'alias watch="watch "' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
exec bash
```

# Install mosquitto_pub
```sh
sudo apt update
sudo apt install mosquitto-clients -y
```

# Create Kubernes cluster
```sh
kind create cluster --config kind.yaml
kubectl get nodes
```

# Install MetalLB
```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
kubectl wait --namespace metallb-system \
             --for=condition=ready pod \
             --selector=app=metallb \
             --timeout=90s
docker network inspect -f '{{.IPAM.Config}}' kind
kubectl apply -f metallb.config
```

# Create iot-stack namespace
```sh
kubectl apply -f namespace.yaml
```

# Create app
```sh
kubectl apply -f mosquitto
kubectl apply -f mariadb
kubectl apply -f telegraf
kubectl apply -f grafana
```

# Change default namespace
```sh
kubectl config set-context --current --namespace=iot-stack
```

# Create database
```sh
kubectl exec -it -n iot-stack mariadb-0 -- mariadb -p
# Enter password "secret"
create database metrics;
exit
```

# Check app status
```sh
kubectl get all
```

# Send MQTT message
```
mosquitto_pub -h 172.18.255.200 -p 8883 --insecure \
--capath /etc/ssl/certs/ \
-u esp32 -P '6&0#GIruG1' \
-t 'home/livingroom/temperature' \
-m 'temperature celsius=20'
```

# Open telegraf logs
```
kubectl logs telegraf-9dbb975b9-57db7 # replace the pod name
```

# Open database
```sh
kubectl exec -it -n iot-stack mariadb-0 -- mariadb -p
# Enter password "secret"
use metrics;
select * from temperature;
exit
```

# Open Grafana
```sh
kubectl port-forward pod/grafana-55757f4bd-6zbvk 3000 # replace the pod name
```