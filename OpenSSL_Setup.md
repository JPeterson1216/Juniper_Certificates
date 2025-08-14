This is a description of the steps taken to create a certificate authority licensing server

This assumes you have a linux distribution server spun up already.
I did this on the latest release of ubuntu server.

1. Install openssl if it is not installed already

`sudo apt update
sudo apt install openssl`

2. Create a directoy Structure

  `mkdir -p ~/myCA/{certs,crl,newcers,private}
  chmod 700 ~/myCA/private
  touch ~/myCA/index.txt
  echo 1000 > ~/myCA/serial`

3. Generate a private key
`openssl ecparam -name secp384r1 -genkey -noout -out ec_key.pem
chmod 400 ~/myCA/private/ec_key.pem`

4. Create a CA Certificate
`openssl req -new -x509 -days 36524 -key private/ec_key.pem -sha384 -out cacert.pem`
You will be prompted to provide information for the certificate. For example:

Country Name: Two-letter country code (e.g., "US")
State or Province Name: Full name of your state
Locality Name: City
Organization Name: Your organization name
Common Name: A name for your CA (e.g., "My CA")

5. Verify the certificate
`openssl x509 -noout -text -in ~/myCA/certs/ca.cert.pem`

6. Edit you config file
openssl uses this file as a config fiel : /etc/ssl/openssl.cnf
`sudo nano /etc/ssl/openssl.cnf`

8. In th config file make sure the directories are correct
![image](https://github.com/user-attachments/assets/c4e3747b-d812-4f17-a5cc-01ce210e7eed)
The dir should point to your "myCA" folder
the certificate should point at your cacert.pem (you could have named it differently)
all of the pointers in the config show above should point to the correct location

9. Set these to optional

![image](https://github.com/user-attachments/assets/e0053d51-9268-454d-8ae3-1fd0673b35f8)

even if that state name is exactly correct - it will say it is different. Haven't figured out why yet

12.  Make these unique - tailor your certificate designated names to match the values put in your CA Cert
![image](https://github.com/user-attachments/assets/41afac26-f0b8-4ee9-aed4-54091f58e9ad)

13.  You can replace the entire openssl config file with something like this :

```
[ ca ]
default_ca = CA_default

[ CA_default ]
dir = ~/myCA
certs = $dir/certs
crl_dir = $dir/crl
new_certs_dir = $dir/newcerts
database = $dir/index.txt
serial = $dir/serial

private_key = $dir/private/ca.key.pem
certificate = $dir/certs/ca.cert.pem
default_md = sha256

policy = policy_strict

[ policy_strict ]
countryName = match
stateOrProvinceName = match
organizationName = match
commonName = supplied

[ req ]
distinguished_name = req_distinguished_name
prompt = no

[ req_distinguished_name ]
countryName = US
stateOrProvinceName = ExampleState
localityName = ExampleCity
organizationName = ExampleOrg
commonName = MyCA
```
 14.  Sign you certificates
 
`sudo openssl ca -in <certificate-ID>.csr -out certs/<certificate-ID>.pem -extfile subalt.txt`

You can specify -days 14610 to make it 40 years.
Keep in mind the validity of your CA certificate. It also needs to be valid for 40 years

create subalt.txt. It will specify the dns of your customer. For multiple domain names. not mandatory
subjectAltName=DNS:<customer-name1>.local
subjectAltName=DNS:<customer-name2>.local

