# Introduction	
Internet of Things (IoT) is a sprawling set of technologies and use cases that has no clear, single definition. One workable view frames IoT as the use of network-connected devices, embedded in the physical environment, to improve some existing process or to enable a new scenario not previously possible.

The Internet of Things (IoT) continues to drive enormous growth in the number of connected systems and devices. The majority of these connected devices use the MQTT protocol to communicate with each other and backend systems. To meet the demands of rapidly expanding networks of devices, MQTT solutions usually require high operational reliability and scalability; be it for vehicles that need mobility services, or machines on the factory floor that require predictive maintenance.

# Architecture Design

![alt text](https://github.com/divc/IOTScaleTest/blob/main/ModernIoTSolution.png)

# Implementation

## Setting up GKE

Following command is used to create regional clusters
```
gcloud container clusters create CLUSTER_NAME \
    --region COMPUTE_REGION \
    --node-locations COMPUTE_ZONE \
    --release-channel CHANNEL
```

## Setting up self managed ASM

Downloading and installing ASM 
```
curl https://storage.googleapis.com/csm-artifacts/asm/install_asm_1.10 > install_asm
curl https://storage.googleapis.com/csm-artifacts/asm/install_asm_1.10.sha256 > install_asm.sha256
sha256sum -c --ignore-missing install_asm.sha256
chmod +x install_asm

./install_asm \
  --project_id PROJECT_ID \
  --cluster_name CLUSTER_NAME \
  --cluster_location CLUSTER_LOCATION \
  --mode install \
  --ca mesh_ca \
  --output_dir DIR_PATH \
  --enable_registration \
  --enable_all
```

Enabling auto-injection of istio sidecars on hivemq namespace
```
kubectl label namespace hivemq istio-injection- istio.io/rev=asm-1102-3 --overwrite
```

## Configuring MQTT and HTTP ports on Istio Ingress Gateway
```
……
  ports:
  - name: hivemq-mqtt-tls-port
    nodePort: 30166
    port: 8883
    protocol: TCP
    targetPort: 8883
  - name: hivemq-mqtt-port
    nodePort: 30893
    port: 1883
    protocol: TCP
    targetPort: 1883
  - name: hivemq-web-port
    nodePort: 30434
    port: 8080
    protocol: TCP
    targetPort: 8080
……..
```

## Installing HiveMQ
Running hivemq deployment 
```
kubectl apply -f hivemq-deployment.yaml 
```

Deploying web gui and mqtt services 
```
kubectl apply -f hivemq-webui-service.yaml
kubectl apply -f hivemq-mqtt-service.yaml
```

## Creating Istio Gateway for MQTT traffic (TCP Mode)
```
kubectl apply -f hivemq-mqtt-gateway.yaml
```


## Creating Istio Gateway for HiveMQ Web HTTP traffic
```
kubectl apply -f hivemq-webui-gateway
```

## Creating Istio Gateway for TLS MQTT traffic (Optional)
Creating self signed certificates 
```
mkdir certs
cd certs

openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=example Inc./CN=example.com' -keyout ca.key -out ca.crt

openssl req -out server.csr -newkey rsa:2048 -nodes -keyout server.key -subj "/CN=httpbin.example.com/O=httpbin organization"
openssl x509 -req -days 365 -CA ca.crt -CAkey ca.key -set_serial 0 -in server.csr -out server.crt
```

Creating secrets in k8s 
```
kubectl create -n istio-system secret tls hivemq-credential --key=server.key --cert=server.crt
```


# Scaling the solution

## Scaling HiveMQ pods
```
kubectl apply -f hivemq-hpa.yaml
```

## Scaling Istio Ingress Gateway pods
```
kubectl apply -f istio-ingressgateway-hpa.yaml
```

# Useful commands 
Subscribing to MQTT 
```
mqtt sub -t topic -h <HOSTNAME> -p <1883/8883> --cert <Certificate Key> --key <Private Key File> --cafile <Root Certificate> 
```