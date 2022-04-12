* [Overview](#overview)
* [Functionality](#functionality)
* [Recommended System Requirements](#recommended-system-requirements)
* [Running Docker Image](#running-docker-image)
* [Configuring Docker Image](#configuring-docker-image)
    - [Storing Data](#storing-data)
    - [Running ONLYOFFICE Community Server on Different Port](#running-onlyoffice-community-server-on-different-port)
    - [Exposing Additional Ports](#exposing-additional-ports)
    - [Running ONLYOFFICE Community Server using HTTPS](#running-onlyoffice-community-server-using-https)
        + [Generation of Self Signed Certificates](#generation-of-self-signed-certificates)
        + [Strengthening the Server Security](#strengthening-the-server-security)
        + [Installation of the SSL Certificates](#installation-of-the-ssl-certificates)
        + [Available Configuration Parameters](#available-configuration-parameters)
* [Installing ONLYOFFICE Community Server integrated with Document and Mail Servers](#installing-onlyoffice-community-server-integrated-with-document-and-mail-servers)
* [Issues](#issues)
    - [Docker Issues](#docker-issues)
    - [Mono Issues](#mono-issues)
* [Project Information](#project-information)
* [User Feedback and Support](#user-feedback-and-support)

## Overview

ONLYOFFICE Community Server is a free open source collaborative system developed to manage documents, projects, customer relationship and email correspondence, all in one place.

## Functionality

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

## Recommended System Requirements

* **RAM**: 4 GB or more
* **CPU**: dual-core 2 GHz or higher
* **Swap file**: at least 2 GB
* **HDD**: at least 2 GB of free space
* **Distributive**: 64-bit Red Hat, CentOS or other compatible distributive with kernel version 3.8 or later, 64-bit Debian, Ubuntu or other compatible distributive with kernel version 3.8 or later
* **Docker**: version 1.4.1 or later

## Running Docker Image

    sudo docker run -i -t -d -p 80:80 onlyoffice/communityserver

This command will install ONLYOFFICE Community Server and all the dependencies it needs.

## Configuring Docker Image
### Storing Data

All the data are stored in the specially-designated directories, **data volumes**, at the following location:
* **/var/log/onlyoffice** for ONLYOFFICE Community Server logs
* **/var/www/onlyoffice/Data** for ONLYOFFICE Community Server data
* **/var/lib/mysql** for MySQL database data

To get access to your data from outside the container, you need to mount the volumes. It can be done by specifying the '-v' option in the docker run command.

    sudo docker run -i -t -d -p 80:80 \
        -v /opt/onlyoffice/Logs:/var/log/onlyoffice  \
        -v /opt/onlyoffice/Data:/var/www/onlyoffice/Data  \
        -v /opt/onlyoffice/MySQL:/var/lib/mysql  onlyoffice/communityserver

Storing the data on the host machine allows you to easily update ONLYOFFICE once the new version is released without losing your data.

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
        -v /opt/onlyoffice/Data:/var/www/onlyoffice/Data  onlyoffice/communityserver

Access to the onlyoffice application can be secured using SSL so as to prevent unauthorized access. While a CA certified SSL certificate allows for verification of trust via the CA, a self signed certificates can also provide an equal level of trust verification as long as each client takes some additional steps to verify the identity of your website. Below the instructions on achieving this are provided.

To secure the application via SSL basically two things are needed:

- **Private key (.key)**
- **SSL certificate (.crt)**

So you need to create and install the following files:

        /opt/onlyoffice/Data/certs/onlyoffice.key
        /opt/onlyoffice/Data/certs/onlyoffice.crt

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

The `/var/www/onlyoffice/Data/` path is the path of the data store, which means that you have to create a folder named certs inside `/opt/onlyoffice/Data/` and copy the files into it and as a measure of security you will update the permission on the `onlyoffice.key` file to only be readable by the owner.

```bash
mkdir -p /opt/onlyoffice/Data/certs
cp onlyoffice.key /opt/onlyoffice/Data/certs/
cp onlyoffice.crt /opt/onlyoffice/Data/certs/
cp dhparam.pem /opt/onlyoffice/Data/certs/
chmod 400 /opt/onlyoffice/Data/certs/onlyoffice.key
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

## Installing ONLYOFFICE Community Server integrated with Document and Mail Servers

ONLYOFFICE Community Server is a part of ONLYOFFICE Free Edition that comprises also Document Server and Mail Server. To install them, follow these easy steps:

**STEP 1**: Installing ONLYOFFICE Document Server.

```bash
sudo docker run -i -t -d  --name onlyoffice-document-server onlyoffice/documentserver
```

**STEP 2**: Installing ONLYOFFICE Mail Server. 

For the mail server correct work you need to specify its hostname 'yourdomain.com'.
To learn more, refer to the [ONLYOFFICE Mail Server documentation](https://github.com/ONLYOFFICE/MailServer "ONLYOFFICE Mail Server documentation").

```bash
sudo docker run --privileged -i -t -d --name onlyoffice-mail-server -p 25:25 -p 143:143 -p 587:587 \
-h yourdomain.com onlyoffice/mailserver
```

**STEP 3**: Installing ONLYOFFICE Community Server

```bash
sudo docker run -i -t -d -p 80:80 -p 5222:5222 -p 443:443 \
--link onlyoffice-mail-server:mail_server \
--link onlyoffice-document-server:document_server \
onlyoffice/communityserver
```

Alternatively, you can use [docker-compose](https://docs.docker.com/compose/install "docker-compose") to install the whole ONLYOFFICE Free Edition at once. For the mail server correct work you need to specify its hostname 'yourdomain.com'. Assuming you have docker-compose installed, execute the following command:

```bash
wget https://raw.githubusercontent.com/ONLYOFFICE/Docker-CommunityServer/master/docker-compose.yml
docker-compose up -d
```

## Issues

### Docker Issues

As a relatively new project Docker is being worked on and actively developed by its community. So it's recommended to use the latest version of Docker, because the issues that you encounter might have already been fixed with a newer Docker release.

The known Docker issue with ONLYOFFICE Document Server with rpm-based distributives is that sometimes the processes fail to start inside Docker container. Fedora and RHEL/CentOS users should try disabling selinux with setenforce 0. If it does not fix the issue, you can either stick with selinux disabled (not recommended by RedHat) or switch to using Ubuntu.

### Mono Issues

ONLYOFFICE installation requires the presence of mono (tested for version 3.12.1 or [older](http://www.mono-project.com/docs/getting-started/install/linux/#accessing-older-releases "older")) that may cause problems for some Linux kernel versions. The full list of supported kernel versions is available [here](http://onlyo.co/1PABPEI "here").


## Project Information

Official website: [http://www.onlyoffice.org](http://onlyoffice.org "http://www.onlyoffice.org")

Code repository: [https://github.com/ONLYOFFICE/CommunityServer](https://github.com/ONLYOFFICE/CommunityServer "https://github.com/ONLYOFFICE/CommunityServer")

License: [GNU GPL v3.0](https://www.gnu.org/copyleft/gpl.html "GNU GPL v3.0")

SaaS version: [http://www.onlyoffice.com](http://www.onlyoffice.com "http://www.onlyoffice.com")


## User Feedback and Support

If you have any problems with or questions about this image, please contact us through a [dev.onlyoffice.org][1].

  [1]: http://dev.onlyoffice.org
