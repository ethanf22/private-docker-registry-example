# DAS Private Docker Registry
A guide to launch a private docker registry secured with Lets Encrypt. 
Tested on Ubuntu 16.04 and Apache 2.4

## Create htpasswd file
For each user you'd like to authenticate, run the following command:
`docker run --entrypoint htpasswd registry:2 -Bbn {user_name} {user_password} > /choose/path/to/htpasswd`
	* Replace `{user_name}` and `{user_password}` for each user you'd like to add.

## Configure Apache
1. Edit the registry.domain.com.conf file
	* Replace `domain` with the name of the domain the registry will be hosted on in the file name
	and 6 places within the file
	* Leave the unsecure ssl paths alone until after you've generated the certificates.
	* Replace `/path/to/file/` with the path to the htpasswd file.
2. Copy the file to the apache sites-available folder, typically `/etc/apache2/sites-available`.
3. Activate the site, `sudo a2ensite registry.{domain}.com.conf`.
4. Restart the server, `sudo service apache2 restart`.
5. Test that the page is served at https://registry.{domain}.com, you should see a default apache page
	after accepting the insecure ssl cert.

## SSL Certificate

### If using Lets Encrypt
1. `sudo git clone https://github.com/certbot/certbot /opt/certbot`
2. `cd /opt/certbot/; ./certbot-auto certonly --apache -d registry.{domain}.com`
3. Your files should be stored in `/etc/letsencrypt/live/{domain}`
4. Navigate to the ssl certificate folder and run the following commands:
	* `sudo cp privkey.pem /path/to/docker/registry/folder/domain.key`
	* `sudo cat cert.pem chain.pem > /path/to/docker/registry/folder/domain.crt`
5. Update your registry.{domain}.com.conf file to link to the new ssl certificates.
	* `SSLCertificateFile /etc/letsencrypt/live/registry.{domain}.com/fullchain.pem`
	* `SSLCertificateKeyFile /etc/letsencrypt/live/registry.{domain}.com/privkey.pem`
	* Uncomment the `Include /etc/letsencrypt/options-ssl-apache.conf`
6. Run `sudo service apache2 restart`.
7. Verify that the url now shows a secured page.

### Anything else
Adapt the above instructions

## Configure Docker
1. Copy the docker-compose.yml file to the server.
2. Run `docker-compose up -d`. If you run into problems, remove the `-d` flag and look for errors.

## Test
1. From your local machine, run `docker login registry.{domain}.com`. Enter your username and password.
2. If you run into errors, look at the return here, the output from the docker container running on the server
and anything in the apache error logs.