# Configure Docker Using TLS Connection
## Install Docker
curl -fsSL https://get.docker.com -o install-docker.sh \
bash install-docker.sh \
usermod -aG docker himel
## Create Cetificates

openssl genrsa -aes256 -out ca-key.pem 4096 \
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem \
openssl genrsa -out server-key.pem 4096 \
openssl req -subj "/CN=192.168.60.10" -sha256 -new -key server-key.pem -out server.csr \
echo subjectAltName = DNS:192.168.60.10,IP:192.168.60.10,IP:127.0.0.1 >> extfile.cnf \
echo subjectAltName = DNS:$HOST,IP:192.168.60.10,IP:127.0.0.1 >> extfile.cnf \
echo extendedKeyUsage = serverAuth >> extfile.cnf \
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
## Create Client Key
openssl genrsa -out key.pem 4096 \
openssl req -subj '/CN=client' -new -key key.pem -out client.csr \
echo extendedKeyUsage = clientAuth > extfile-client.cnf \
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile-client.cnf
## Docker Configaration To run on tls
cp -v {ca,cert,key}.pem ~/.docker \
sudo systemctl stop docker \
sudo mkdir /etc/systemd/system/docker.service.d \
sudo vim /etc/systemd/system/docker.service.d/override.conf

[Service] \
ExecStart= \
ExecStart=/usr/bin/dockerd -D -H unix:///var/run/docker.sock --tlsverify --tlscacert=/home/himel/.docker/ca.pem --tlscert=/home/himel/.docker/server-cert.pem --tlskey=/home/himel/.docker/server-key.pem -H tcp://0.0.0.0:2376

systemctl daemon-reload \
systemctl restart docker \
systemctl status docker.service 
## For Docker Client
mkdir ~/.docker
## copy key.pam,cert.pam,ca.pam file to client 
export DOCKER_HOST="tcp://IP:2376"
# Jenkins Configaration
<img width="661" alt="image" src="https://github.com/user-attachments/assets/4ba4913c-a635-4207-a9a2-bed213c5ef59" />

### Agent Configaration in UI
In container mount in agent \
type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
type=bind,src=/home/himel/agent-jenkins/cert,dst=/home/jenkins/cert \
<img width="677" alt="image" src="https://github.com/user-attachments/assets/e400728b-6e64-4056-9da1-5870e4c548e6" />

### Remote File System Root
/home/jenkins/agent \
<img width="677" alt="image" src="https://github.com/user-attachments/assets/c6cb04a4-58e0-4086-9551-d0c150a5cf52" />

### Login to agent using agent ssh credential 

<img width="668" alt="image" src="https://github.com/user-attachments/assets/06488f41-a4f9-4f04-8dea-aa8f65abd93b" />


### in environment in jenkinsfile 
DOCKER_TLS_VERIFY=1 \
export DOCKER_CERT_PATH=/home/jenkins/cert \
export DOCKER_HOST=tcp://192.168.60.10:2376
### Demo Jenkins initiation sample
pipeline {
    agent {
        label 'agent'
    }
    environment {
        DOCKER_TLS_VERIFY = '1'
        DOCKER_CERT_PATH = '/home/jenkins/cert'
        DOCKER_HOST = 'tcp://192.168.60.10:2376'
    
    }


