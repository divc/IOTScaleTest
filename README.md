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