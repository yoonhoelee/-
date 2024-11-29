I'll help walk you through the process of updating SSL certificates on these Apache servers. Based on the image, I can see these are Apache 2.2.27 servers with different roles (NOTE servers, BIZ servers, MTS servers, etc.).

Here's a detailed step-by-step process:

1. First, back up the existing certificates:
```bash
# For each server, backup the current certificates
cp /conf/ssl/cert.pem /conf/ssl/cert.pem.backup
cp /conf/ssl/DigiCertCA.pem /conf/ssl/DigiCertCA.pem.backup
```

2. Prepare the new certificates on your local machine. You'll need:
   - The new SSL certificate (cert.pem)
   - The new CA certificate chain (DigiCertCA.pem or Thawte-Chain_sha2.pem depending on server)

3. Transfer the new certificates to each server. Using SCP:
```bash
scp new_cert.pem username@[SERVER_IP]:/tmp/
scp new_chain.pem username@[SERVER_IP]:/tmp/
```

4. For each server, install the new certificates:
```bash
# Move certificates to proper location
sudo mv /tmp/new_cert.pem /conf/ssl/cert.pem
sudo mv /tmp/new_chain.pem /conf/ssl/DigiCertCA.pem  # or Thawte-Chain_sha2.pem for MTS servers
```

5. Verify certificate permissions:
```bash
sudo chown root:root /conf/ssl/*.pem
sudo chmod 600 /conf/ssl/*.pem
```

6. Test the Apache configuration:
```bash
/home1/irteam/apps/apache-2.2.27/bin/apachectl configtest
```

7. Restart Apache to apply the changes:
```bash
/home1/irteam/apps/apache-2.2.27/bin/apachectl graceful
```

8. Verify the new certificate is working:
```bash
openssl s_client -connect [SERVER_IP]:443 -servername [SERVER_NAME]
```

Important considerations:
- Perform this during a maintenance window
- Test the process on one server first (perhaps cotbtap01 as it appears to be a test server)
- Have a rollback plan ready using the backup certificates
- Remember different servers use different certificate chains (DigiCert vs Thawte)
- Consider automating this process if you need to do it frequently

Would you like me to provide more detail about any of these steps or explain how to verify the certificate details before installation?
