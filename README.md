# TESTE de Geração de certificado

https://www.freecodecamp.org/news/how-to-get-https-working-on-your-local-development-environment-in-5-minutes-7af615770eec/

https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html

## The following commands are needed to create a root Certificate Authority:

openssl genrsa -des3 -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024  -out rootCA.pem

## Import root certificate to windows registry 

certutil –addstore -enterprise –f "Root" rootCA.pem

### remove certificate
certutil –delstore -enterprise –f "Root" rootCA.pem


## The following commands are needed to create an SSL certificate issued by the self created root Certificate Authority:

openssl req -new -nodes -out ./server/server.csr -newkey rsa:2048 -keyout ./server/server.key

<!--$OPENSSL ca -config $CA_PATH/caconfig.cnf -passin pass:123456 -in $CONFS_DIR/servidorreq.pem -out $CONFS_DIR/servidor_crt.pem &&-->

openssl x509 -req -in ./server/server.csr -CA ./rootCA/rootCA.pem -CAkey ./rootCA/rootCA.key -CAcreateserial -out ./server/server.pem -days 1024 -sha256 -extfile ./server/v3.ext

## The referenced v3.ext file should look something like this:

authorityKeyIdentifier=keyid,issuer  
basicConstraints=CA:FALSE  
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment  
subjectAltName = @alt_names  
[alt_names]  
DNS.1 = localhost  
DNS.2 = demobackend-site.dev  
DNS.3 = demobackend-rest.dev  

## Concat server certificate + rootCA

Ordem de concatenação correta:
 1. Chaves
 2. Certificado
 3. CA Intermediaria
 4. CA raiz

cat ./server/server.pem ./rootCA/rootCA.pem > ./server/serverChain.cer

## In order to bundle the server certificate and private key into a single file the following command needs to be executed:

Certificado completo (chave + certificado + CA), este formato serve para importacao via portecle (import key pair):
openssl pkcs12 -export -in ./server/server.pem -passin pass:123456 -passout pass:123456 -inkey ./server/server.key -chain -CAfile ./server/serverChain.cer -out ./server/server.pkcs12

openssl pkcs12 -inkey ./server/server.key -in ./server/server.pem -export -out ./server/server.pfx

