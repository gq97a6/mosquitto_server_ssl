# How to implement SSL for mosquitto server (self-signed)

###### Get openssl:
```
sudo apt install openssl
```

###### Change directory to where your mosquitto server is located:
```
cd /etc/mosquitto
```

###### Create temporary folder:
```
mkdir tmp && cd tmp
```

Copy ***ca.conf*** and ***server.conf*** files from this repo there, replace every occurrence of ***???*** with your configuration

###### Create certificate authority key and certificate pair:
```
openssl genrsa -des3 -out ca.key 2048 && openssl req -new -x509 -days 1826 -key ca.key -out ca.crt -config ca.conf
```

###### Create server key and certificate signing request for server:
```
openssl genrsa -out server.key 2048 && openssl req -new -out server.csr -key server.key -config server.conf
```

###### Create server certificate:
```
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 360 -extfile server.conf -extensions v3_req
```

###### Remove redundant .csr and .srl files.
```
rm *.csr && rm *.srl
```

###### Change ownership of files to user running server:
```
chown -R mosquitto:mosquitto ./
```

Transfer ***ca.crt*** to ***ca_certificates*** folder, ***server.crt*** and ***server.key*** to ***certs*** folder.

###### Change directory to where your mosquitto server configuration is located:
```
cd /etc/mosquitto/conf.d
```

###### Create secure.conf ssl config e.g. :
```
listener 8883
cafile /etc/mosquitto/ca_certificates/ca.crt
keyfile /etc/mosquitto/certs/server.key
certfile /etc/mosquitto/certs/server.crt
```

###### Change ownership of secure.conf to user running server:
```
chown mosquitto:mosquitto secure.conf
```

Save ***ca.key*** and config files for future key and certificate pairs creation.

###### Restart mosquitto server, repeat if necessary:
```
systemctl restart mosquitto
```

###### Check if server is running properly:
```
systemctl status mosquitto
```

Use ***ca.crt*** certificate to identify this server on a client side.