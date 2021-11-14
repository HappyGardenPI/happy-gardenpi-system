# Happy GardenPI system
Thsi project contain all data fo build theredfore configure Happi GardenPI

## Hardware depenencies
* 1 x [RaspberryPI Zero W](https://www.raspberrypi.org/products/raspberry-pi-zero-w/) or RaspberryPI Zero WH
* 1 x [Sd card minimum form 8gb ](https://www.raspberrypi.org/documentation/installation/sd-cards.md)
* 1 x [LCD 1602](https://www.amazon.it/HiLetgo-HD44780-Character-Blacklight-Raspberry/dp/B079KF812H/ref=pd_sbs_25/262-6602057-3019947?pd_rd_w=YUofM&pf_rd_p=73021335-3337-4067-9dba-3816378c8630&pf_rd_r=B5E88M3TWGAAJS70EH8R&pd_rd_r=83fe42d7-d6e8-43d2-bdb2-6a6d27cec734&pd_rd_wg=zZQsb&pd_rd_i=B079KF812H&psc=1)
* 1 x [Elegoo 4 Channel DC 5V Relay](https://www.amazon.it/gp/product/B06XRJ6XBJ)

## Install OS
[Installing operating system images](https://www.raspberrypi.org/documentation/installation/installing-images) by [Raspberry Pi Imager](https://www.raspberrypi.org/software/) then install [Raspberry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/#raspberry-pi-os-32-bit)

## Install packages needed in Raspeberry PI Zero W
```
apt install mosquitto mosquitto-clients gdbserver 
```

### Configure Mosquitto MQTT broker
Congioure Mosquitto fop TSL connnections.  

Set user name and password for pub new messages:
```
sudo mosquitto_passwd -c /etc/mosquitto/passwd hgardenpi
```
only for testing I use user hgardenpi whit same password. 

Now set configuration:
```
sudo nano /etc/mosquitto/conf.d/default.conf
```
This should open an empty file. Paste in the following:
```
listener 1883
allow_anonymous false
password_file /etc/mosquitto/passwd
```
Now we need to restart Mosquitto and test our changes.
```
sudo systemctl restart mosquitto
```

#### Create Mosquitto certificate:
```
openssl req -new -x509  -extensions v3_ca -keyout mosquitto.ca.key -out mosquitto.ca.crt
```
Insert password greater then 4 chars, in my case only for test purpose I use hgardenpi. 

Then insert key password for create cert adn any other data for example:
```
Enter pass phrase for mosquitto.ca.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:IT 
State or Province Name (full name) [Some-State]:Modena
Locality Name (eg, city) []:Marano sul Panaro
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Happy GardenPI
Organizational Unit Name (eg, section) []:IT
Common Name (e.g. server FQDN or YOUR name) []:passy-raspberrypi
Email Address []:happy.gardenpi@zresa.it    
```
Now we have a certificate. We can create the keys and certificates of the broker:

A private key is generated (without a password):
```
openssl genrsa -out mosquitto.key 2048
```
Now, we create a signing request file from this key:
```
openssl req -out mosquitto.csr -key mosquitto.key -new
```
Now we can pass the Certificate Signing Request (csr) file to our validation authority:
```
openssl x509 -req -in mosquitto.csr -CA mosquitto.ca.crt -CAkey mosquitto.ca.key -CAcreateserial -out mosquitto.crt 
```

#### Create Clients certificate:
We’re practically doing the same thing again, this time to certify a client:
```
openssl genrsa -out client.key 2048
openssl req -out client.csr -key client.key -new
openssl x509 -req -in client.csr -CA mosquitto.ca.crt -CAkey mosquitto.ca.key -CAcreateserial -out client.crt 
```
Now set configuration:
```
sudo nano /etc/mosquitto/conf.d/default.conf
```
This should open an empty file. Paste in the following:
```
listener 1883 localhost

listener 8883
cafile /path_of_certificates/mosquitto.ca.crt

# Path to the PEM encoded server certificate.
certfile /path_of_certificates/mosquitto.crt

# Path to the PEM encoded keyfile.
keyfile /path_of_certificates/mosquitto.key
require_certificate true
```


