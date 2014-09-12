For reference, these are the steps I undertook to update the smart-workforce certificate which is held in a Java Keystore:

(Note, keystore password is the same as that of the db.)

```bash
# convert new cert to X.509
openssl x509 -in public.pem -out public.crt
# copy to server
scp public.crt keaton:
ssh keaton
cp /etc/.keystore.ssplus-ws ~
# list certs
keytool -list -keystore .keystore.ssplus-ws
# delete old cert
keytool -delete -alias smart-workforce -keystore .keystore.ssplus-ws
# import new cert
keytool -import -trustcacerts -alias smart-workforce -file public.crt -keystore .keystore.ssplus-ws
# replace keystore (advisable to backup first)
sudo mv .keystore.ssplus-ws /etc/
# restart tomcat
```

The keystore also needs to be placed in the same directory on europa on spike.