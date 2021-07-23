Whitelist a host in Disconnected Cluster
How to ssh into disconnected cluster proxy host
1. Click on the link - https://github.com/openshift/shared-secrets/blob/master/aws/openshift-qe.pem
2. Click Raw option of openshift-qe.pem file
3. Open terminal and execute
4. wget <copy the URL from Raw view of openshift-qe.pem file> -O ~/.ssh/openshift-qe.pem
5. chmod 400 ~/.ssh/openshift-qe.pem
6. ssh -i ~/.ssh/openshift-qe.pem ec2-user@<HOST>
Use squid.config file /srv/squid/etc/squid.conf for whitelisting urls.
To restart squid service - $ sudo systemctl restart squid-proxy
