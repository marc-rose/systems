# systems

Creating a request using the openssl command-line utility:
The following command performs the following actions
- create new certificate request ssl.csl
- create private key ssl.key
- target sslconfig.txt for detailed configuration (country, locality, etc.)

openssl req -new -sha256 -nodes -out ssl.csr -newkey rsa:2048 -keyout ssl.key -config sslconfig.txt
