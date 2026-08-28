# crAPI Notes

## Target URLs
http://crapi.lab:8888 http://crapi.apisec.ai
http://crapi.lab:8025

## Credentials
nonso@mail.com:Password123!
nonso1@mail.com:Password123!

## Reverse Engineering Target Website
### Start Man in the Middle Web Server
```bash
$ mitmweb
```
After running the web application then convert the flow file to `Open API Specification` 

```bash
$ mitmproxy2swagger -i flow -o crapi_spec.yaml -p http://crapi.lab:8888 -f flow
```
### 
