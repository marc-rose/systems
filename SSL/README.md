# Creating a request using the openssl command-line utility
The following command performs the following actions:
- create new certificate request ssl.csr
- create private key ssl.key
- target sslconfig.txt for detailed configuration (country, locality, etc.)

Modify the `sslconfig.txt` file before running this command.

`openssl req -new -sha256 -nodes -out ssl.csr -newkey rsa:2048 -keyout ssl.key -config sslconfig.txt`

The CSR created from this command can be submitted to a CA to be signed. After the request is signed you will be given a signed certificate.
Several attributes of the certificate may be changed in the signed certificate, depending on the configuration of the CA.
Certificate Authorities (depending on the cert template/product) may not allow for Subject Alternate Names (SANs) to be specified.