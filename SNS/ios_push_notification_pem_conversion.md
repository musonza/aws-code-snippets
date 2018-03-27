# iOS Push Notification PEM Conversion

## Required:
- .cer certificate file
- .p12 private key file
- openssl

## Steps:
1. Convert .cer to .pem with the following command: (afterwards you may be prompted for an import password)
```bash
openssl x509 -in myapnsappcert.cer -inform DER -out myapnsappcert.pem
```
2. Convert .p12 to .pem with the following command:
```bash
openssl pkcs12 -in myapnsappprivatekey.p12 -out myapnsappprivatekey.pem -nodes -clcerts
```
3. Verify the certificate and private key with the following command: (remove sandbox for production)
```bash
openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert myapnsappcert.pem -key myapnsappprivatekey.pem
```
