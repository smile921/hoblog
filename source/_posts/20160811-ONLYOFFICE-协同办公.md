---
title: ONLYOFFICE 协同办公
date: 2016-08-11 11:28:16
tags:
  - office
  - docker
---

## onlyOffice 
### Collaborative system for managing documents, projects, customer relations and emails in one place
* 在线编辑文档
* 客户关系管理
* 邮件服务
* 文档管理
* 项目管理
* ONLYOFFICE Community Server is a free open source collaborative system developed to manage documents, projects, customer relationship and email correspondence, all in one place.

* Cross platform solution: Linux, Windows
* Document management
* Integration with Google Drive, Box, Dropbox, OneDrive, OwnCloud
* File sharing
* Document embedding
* Access rights management
* Customizable CRM
* Web-to-lead form
* Invoicing system
* Project Management
* Gantt Chart
* Milestones, task dependencies and subtasks
* Time tracking
* Automated reports
* Blogs, forums, polls, wiki
* Calendar
* Email Aggregator
* People module (employee database)
* Instant Messenger
* Support of more than 20 languages


## Running Docker Image

    sudo docker run -i -t -d -p 80:80 onlyoffice/communityserver

This command will install ONLYOFFICE Community Server and all the dependencies it needs.

## Configuring Docker Image

###Auto-restart

To make Docker auto-restart containers on reboot, please use the --restart=always in the docker run command:

	sudo docker run -i -t -d -p 80:80 --restart=always onlyoffice/communityserver

### Storing Data

All the data are stored in the specially-designated directories, **data volumes**, at the following location:
* **/var/log/onlyoffice** for ONLYOFFICE Community Server logs
* **/var/www/onlyoffice/Data** for ONLYOFFICE Community Server data
* **/var/lib/mysql** for MySQL database data

To get access to your data from outside the container, you need to mount the volumes. It can be done by specifying the '-v' option in the docker run command.

    sudo docker run -i -t -d -p 80:80 \
        -v /app/onlyoffice/CommunityServer/logs:/var/log/onlyoffice  \
        -v /app/onlyoffice/CommunityServer/data:/var/www/onlyoffice/Data  \
        -v /app/onlyoffice/CommunityServer/mysql:/var/lib/mysql  onlyoffice/communityserver

Storing the data on the host machine allows you to easily update ONLYOFFICE once the new version is released without losing your data.

### Using MySQL
ONLYOFFICE uses **MySQL 5.5** to store its data.

By default the MySQL server is started internally. But you can easily configure the image to use an external MySQL database. 

If you have an external MySQL server installed on your machine, execute the following command:

```bash
	sudo docker run -i -t -d -p 80:80 -p 5222:5222 -p 443:443 \
	-e MYSQL_SERVER_HOST="127.0.0.1" \
	-e MYSQL_SERVER_PORT="3306" \
	-e MYSQL_SERVER_DB_NAME="onlyoffice" \
	-e MYSQL_SERVER_USER="usr_onlyoffice" \
	-e MYSQL_SERVER_PASS="onlyoffice123" \
	onlyoffice/communityserver
```

### Running ONLYOFFICE Community Server on Different Port

To change the port, use the -p command. E.g.: to make your portal accessible via port 8080 execute the following command:

    sudo docker run -i -t -d -p 8080:80 onlyoffice/communityserver

### Exposing Additional Ports

The container ports to be exposed for **incoming connections** are the folloing:

* **80** for plain HTTP
* **443** when HTTPS is enabled (see below)
* **5222** for XMPP-compatible instant messaging client (for ONLYOFFICE Talk correct work)

You can expose ports by specifying the '-p' option in the docker run command.

    sudo docker run -i -t -d -p 80:80  -p 443:443  -p 5222:5222   onlyoffice/communityserver

For **outgoing connections** you need to expose the following ports:

* **80** for HTTP
* **443** for HTTPS

Additional ports to be exposed for the mail client correct work:

* **25** for SMTP
* **465** for SMTPS
* **143** for IMAP
* **993** for IMAPS
* **110** for POP3
* **995** for POP3S

### Running ONLYOFFICE Community Server using HTTPS

        sudo docker run -i -t -d -p 80:80  -p 443:443 \
        -v /app/onlyoffice/CommunityServer/data:/var/www/onlyoffice/Data  onlyoffice/communityserver

Access to the onlyoffice application can be secured using SSL so as to prevent unauthorized access. While a CA certified SSL certificate allows for verification of trust via the CA, a self signed certificates can also provide an equal level of trust verification as long as each client takes some additional steps to verify the identity of your website. Below the instructions on achieving this are provided.

To secure the application via SSL basically two things are needed:

- **Private key (.key)**
- **SSL certificate (.crt)**

So you need to create and install the following files:

        /app/onlyoffice/CommunityServer/data/certs/onlyoffice.key
        /app/onlyoffice/CommunityServer/data/certs/onlyoffice.crt

When using CA certified certificates, these files are provided to you by the CA. When using self-signed certificates you need to generate these files yourself. Skip the following section if you are have CA certified SSL certificates.

#### Generation of Self Signed Certificates

Generation of self-signed SSL certificates involves a simple 3 step procedure.

**STEP 1**: Create the server private key

```bash
openssl genrsa -out onlyoffice.key 2048
```

**STEP 2**: Create the certificate signing request (CSR)

```bash
openssl req -new -key onlyoffice.key -out onlyoffice.csr
```

**STEP 3**: Sign the certificate using the private key and CSR

```bash
openssl x509 -req -days 365 -in onlyoffice.csr -signkey onlyoffice.key -out onlyoffice.crt
```

You have now generated an SSL certificate that's valid for 365 days.

#### Strengthening the server security

This section provides you with instructions to [strengthen your server security](https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html).
To achieve this you need to generate stronger DHE parameters.

```bash
openssl dhparam -out dhparam.pem 2048
```

#### Installation of the SSL Certificates

Out of the four files generated above, you need to install the `onlyoffice.key`, `onlyoffice.crt` and `dhparam.pem` files at the onlyoffice server. The CSR file is not needed, but do make sure you safely backup the file (in case you ever need it again).

The default path that the onlyoffice application is configured to look for the SSL certificates is at `/var/www/onlyoffice/Data/certs`, this can however be changed using the `SSL_KEY_PATH`, `SSL_CERTIFICATE_PATH` and `SSL_DHPARAM_PATH` configuration options.

The `/var/www/onlyoffice/Data/` path is the path of the data store, which means that you have to create a folder named certs inside `/app/onlyoffice/CommunityServer/data/` and copy the files into it and as a measure of security you will update the permission on the `onlyoffice.key` file to only be readable by the owner.

```bash
mkdir -p /app/onlyoffice/CommunityServer/data/certs
cp onlyoffice.key /app/onlyoffice/CommunityServer/data/certs/
cp onlyoffice.crt /app/onlyoffice/CommunityServer/data/certs/
cp dhparam.pem /app/onlyoffice/CommunityServer/data/certs/
chmod 400 /app/onlyoffice/CommunityServer/data/certs/onlyoffice.key
```

You are now just one step away from having our application secured.

#### Available Configuration Parameters

*Please refer the docker run command options for the `--env-file` flag where you can specify all required environment variables in a single file. This will save you from writing a potentially long docker run command.*

Below is the complete list of parameters that can be set using environment variables.

- **ONLYOFFICE_HTTPS_HSTS_ENABLED**: Advanced configuration option for turning off the HSTS configuration. Applicable only when SSL is in use. Defaults to `true`.
- **ONLYOFFICE_HTTPS_HSTS_MAXAGE**: Advanced configuration option for setting the HSTS max-age in the onlyoffice nginx vHost configuration. Applicable only when SSL is in use. Defaults to `31536000`.
- **SSL_CERTIFICATE_PATH**: The path to the SSL certificate to use. Defaults to `/var/www/onlyoffice/Data/certs/onlyoffice.crt`.
- **SSL_KEY_PATH**: The path to the SSL certificate's private key. Defaults to `/var/www/onlyoffice/Data/certs/onlyoffice.key`.
- **SSL_DHPARAM_PATH**: The path to the Diffie-Hellman parameter. Defaults to `/var/www/onlyoffice/Data/certs/dhparam.pem`.
- **SSL_VERIFY_CLIENT**: Enable verification of client certificates using the `CA_CERTIFICATES_PATH` file. Defaults to `false`
- **MYSQL_SERVER_HOST**: The IP address or the name of the host where the server is running.
- **MYSQL_SERVER_PORT**: The port number.
- **MYSQL_SERVER_DB_NAME**: The name of a MySQL database to be created on image startup.
- **MYSQL_SERVER_USER**: The new user name with superuser permissions for the MySQL account.
- **MYSQL_SERVER_PASS**: The password set for the MySQL account. 

## Installing ONLYOFFICE Community Server integrated with Document and Mail Servers

ONLYOFFICE Community Server is a part of ONLYOFFICE Community Edition that comprises also Document Server and Mail Server. To install them, follow these easy steps:

**STEP 1**: Create the 'onlyoffice' network.

```bash
docker network create --driver bridge onlyoffice
```
Than launch containers on it using the 'docker run --net onlyoffice' option:

**STEP 1**: Install ONLYOFFICE Document Server.

```bash
sudo docker run --net onlyoffice -i -t -d --restart=always --name onlyoffice-document-server \
	-v /app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data \
	-v /app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice \
	onlyoffice/documentserver
```

**STEP 2**: Install ONLYOFFICE Mail Server. 

For the mail server correct work you need to specify its hostname 'yourdomain.com'.
To learn more, refer to the [ONLYOFFICE Mail Server documentation](https://github.com/ONLYOFFICE/Docker-MailServer "ONLYOFFICE Mail Server documentation").

```bash
sudo docker run --net onlyoffice --privileged -i -t -d --restart=always --name onlyoffice-mail-server \
	-p 25:25 -p 143:143 -p 587:587 \
	-v /app/onlyoffice/MailServer/data:/var/vmail \
	-v /app/onlyoffice/MailServer/data/certs:/etc/pki/tls/mailserver \
	-v /app/onlyoffice/MailServer/logs:/var/log \
	-v /app/onlyoffice/MailServer/mysql:/var/lib/mysql \
	-h yourdomain.com \
	onlyoffice/mailserver
```

**STEP 3**: Install ONLYOFFICE Community Server

```bash
sudo docker run --net onlyoffice -i -t -d --restart=always --name onlyoffice-community-server \
	-p 80:80 -p 5222:5222 -p 443:443 \
	-v /app/onlyoffice/CommunityServer/data:/var/www/onlyoffice/Data \
	-v /app/onlyoffice/CommunityServer/mysql:/var/lib/mysql \
	-v /app/onlyoffice/CommunityServer/logs:/var/log/onlyoffice \
	-v /app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/DocumentServerData \
	-e DOCUMENT_SERVER_PORT_80_TCP_ADDR=onlyoffice-document-server \
	-e MAIL_SERVER_DB_HOST=onlyoffice-mail-server \
	onlyoffice/communityserver
```

Alternatively, you can use an automatic installation script to install the whole ONLYOFFICE Community Edition at once. For the mail server correct work you need to specify its hostname 'yourdomain.com'.

**STEP 1**: Download the Community Edition Docker script file

```bash
wget http://download.onlyoffice.com/install/opensource-install.sh
```

**STEP 2**: Install ONLYOFFICE Community Edition executing the following command:

```bash
bash opensource-install.sh -md yourdomain.com
```

Or, use [docker-compose](https://docs.docker.com/compose/install "docker-compose"). For the mail server correct work you need to specify its hostname 'yourdomain.com'. Assuming you have docker-compose installed, execute the following command:

```bash
wget https://raw.githubusercontent.com/ONLYOFFICE/Docker-CommunityServer/master/docker-compose.yml
docker-compose up -d
```

## Upgrading ONLYOFFICE Community Server

To upgrade to a newer release, please follow there easy steps:

**STEP 1**: Make sure that all the container volumes are mounted following the **Storing Data** section instructions:
 
	sudo docker inspect --format='{{range $p,$conf:=.HostConfig.Binds}}{{$conf}};{{end}}' {{COMMUNITY_SERVER_ID}} 

where
	{{COMMUNITY_SERVER_ID}} stands for a container name or ID

**STEP 2** Remove the current container
	sudo docker rm -f {{COMMUNITY_SERVER_ID}}

**STEP 3** Remove the current image
	sudo docker rmi -f $(sudo docker images | grep onlyoffice/communityserver | awk '{ print $3 }')

**STEP 4** Run the new image with the same map paths

	sudo docker run -i -t -d -p 80:80 \
	-v /app/onlyoffice/CommunityServer/logs:/var/log/onlyoffice  \
	-v /app/onlyoffice/CommunityServer/data:/var/www/onlyoffice/Data  \
	-v /app/onlyoffice/CommunityServer/mysql:/var/lib/mysql  onlyoffice/communityserver


## Project Information

Official website: [http://www.onlyoffice.org](http://onlyoffice.org "http://www.onlyoffice.org")

Code repository: [https://github.com/ONLYOFFICE/CommunityServer](https://github.com/ONLYOFFICE/CommunityServer "https://github.com/ONLYOFFICE/CommunityServer")
