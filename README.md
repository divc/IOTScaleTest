# Introduction	
Internet of Things (IoT) is a sprawling set of technologies and use cases that has no clear, single definition. One workable view frames IoT as the use of network-connected devices, embedded in the physical environment, to improve some existing process or to enable a new scenario not previously possible.

The Internet of Things (IoT) continues to drive enormous growth in the number of connected systems and devices. The majority of these connected devices use the MQTT protocol to communicate with each other and backend systems. To meet the demands of rapidly expanding networks of devices, MQTT solutions usually require high operational reliability and scalability; be it for vehicles that need mobility services, or machines on the factory floor that require predictive maintenance.

# Architecture Design

![alt text](https://github.com/divc/IOTScaleTest/main/ModernIoTSolution.png)

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

## Configuring MQTT and HTTP ports on Istio Ingress Gateway
## Installing HiveMQ


3.5 Creating Istio Gateway for MQTT traffic (TCP Mode)
3.6 Creating Istio Gateway for HiveMQ Web HTTP traffic
3.7 Creating Istio Gateway for TLS MQTT traffic (Optional)
3.8 Validating the setup
3.9 Dry run of the setup
4. Scaling the solution
4.1 Scaling HiveMQ pods
4.2 Scaling Istio Ingress Gateway pods
4.3 Scaling MQTT Client for 1MM+ Concurrent Connections per second
4.3.1 JMeter Installation
4.3.2 MQTT JMeter Installation
4.3.3 Setting MQTT Samplers for 1MM+ Connections
4.3.4 Setting TLS MQTT Samplers
4.3.5 Creating Thread Pools for 1MM+ Connections
5. Final Thoughts








Download self managed ASM script -- 

curl https://storage.googleapis.com/csm-artifacts/asm/install_asm_1.10 > install_asm

Download SHA-256 

curl https://storage.googleapis.com/csm-artifacts/asm/install_asm_1.10.sha256 > install_asm.sha256

Check SHA-256

sha256sum -c --ignore-missing install_asm.sha256

Make it executable  

chmod +x install_asm

Validate ASM installation with --mode install and Workload Identity enabled 

./install_asm \
  --project_id PROJECT_ID \
  --cluster_name CLUSTER_NAME \
  --cluster_location CLUSTER_LOCATION \
  --mode install \
  --output_dir DIR_PATH \
  --only_validate

Installing ASM with default features and mode –install 

./install_asm \
  --project_id PROJECT_ID \
  --cluster_name CLUSTER_NAME \
  --cluster_location CLUSTER_LOCATION \
  --mode install \
  --ca mesh_ca \
  --output_dir DIR_PATH \
  --enable_registration \
  --enable_all


kubectl label namespace hivemq istio-injection- istio.io/rev=asm-1102-3 --overwrite


https://www.blazemeter.com/blog/how-use-counter-jmeter-test


Creating certs 




mkdir ca
cd ca
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout ca.key -out ca.crt

cd ..
mkdir broker
cd broker
openssl req -out broker.csr -newkey rsa:2048 -nodes -keyout broker.key


openssl x509 -req -days 365 -CA ../ca/ca.crt -CAkey ../ca/ca.key -set_serial 0 -in broker.csr -out broker.crt

kubectl create -n hivemq secret tls hivemq-credential --key=broker.key --cert=broker.crt

cd ..
mkdir client 
cd client 

openssl req -out client.csr -newkey rsa:2048 -nodes -keyout client.key

openssl x509 -req -days 365 -CA ../ca/ca.crt -CAkey ../ca/ca.key -set_serial 0 -in client.csr -out client.crt



kubectl create -n istio-system secret tls hivemq-credential --key=broker.key --cert=broker.crt