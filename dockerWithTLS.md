# Configure Docker Using TLS Connection
# Install Docker
curl -fsSL https://get.docker.com -o install-docker.sh
bash install-docker.sh
usermod -aG docker himel
## Create Cetificates
openssl genrsa -aes256 -out ca-key.pem 4096 \
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem \
openssl genrsa -out server-key.pem 4096 \
openssl req -subj "/CN=192.168.60.10" -sha256 -new -key server-key.pem -out server.csr \
echo subjectAltName = DNS:192.168.60.10,IP:192.168.60.10,IP:127.0.0.1 >> extfile.cnf \
echo subjectAltName = DNS:$HOST,IP:192.168.60.10,IP:127.0.0.1 >> extfile.cnf \
echo extendedKeyUsage = serverAuth >> extfile.cnf \
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf \
## Create Client Key

openssl genrsa -out key.pem 4096
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
echo extendedKeyUsage = clientAuth > extfile-client.cnf
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile-client.cnf

## Docker Configaration To run on tls
cp -v {ca,cert,key}.pem ~/.docker <br />
sudo systemctl stop docker <br />
sudo mkdir /etc/systemd/system/docker.service.d <br />
sudo vim /etc/systemd/system/docker.service.d <br />

[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -D -H unix:///var/run/docker.sock --tlsverify --tlscacert=/home/himel/.docker/ca.pem --tlscert=/home/himel/.docker/server-cert.pem --tlskey=/home/himel/.docker/server-key.pem -H tcp://0.0.0.0:2376

systemctl daemon-reload <br />
systemctl restart docker <br />
systemctl status docker.service <br />
## For Docker Client
mkdir ~/.docker <br />
#### copy key.pam,cert.pam,ca.pam file to client <br />
export DOCKER_HOST="tcp://IP:2376" <br />


