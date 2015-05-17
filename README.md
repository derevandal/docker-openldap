# osixia/openldap

A docker image to run OpenLDAP.
> [www.openldap.org](http://www.openldap.org/)

Fork of Nick Stenning docker-slapd :
https://github.com/nickstenning/docker-slapd

Add support of tls. Use docker 1.5.0

## Quick start
Run OpenLDAP docker image :

	docker run -d osixia/openldap
  
This start a new container with a OpenLDAP server running inside.
The odd string printed by this command is the `CONTAINER_ID`.
We are going to use this `CONTAINER_ID` to execute some commands inside the container.

Wait 1 or 2 minutes the container startup to be completed.

Then run a terminal on this container,
make sure to replace `CONTAINER_ID` by your container id :

	docker exec -it CONTAINER_ID bash

You should now be in the container terminal, 
and we can search on the ldap server :
	
	ldapsearch -x -h 127.0.0.1 -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin
	
This should output :

	# extended LDIF
	#
	# LDAPv3
	# base <dc=example,dc=org> with scope subtree
	# filter: (objectclass=*)
	# requesting: ALL
	#
	
	[...]

	# numResponses: 3
	# numEntries: 2
	
if you have the following error, OpenLDAP is not started yet, wait some time.

		ldap_sasl_bind(SIMPLE): Can't contact LDAP server (-1)
	
	
## Examples

### Create new ldap server

This is the default behaviour when you run the image.
It will create an empty ldap for the compagny **Example Inc.** and the domain **example.org**.

By default the admin has the password **admin**. All those default settings can be changed at the docker command line, for example :

	docker run -e LDAP_ORGANISATION="My Compagny" -e LDAP_DOMAIN="my-compagny.com" \
	-e LDAP_ADMIN_PASSWORD="JonSn0w" -d osixia/openldap

#### Data persitance

The directories `/var/lib/ldap` (LDAP database files) and `/etc/ldap/slapd.d`  (LDAP config files) has been declared as volumes, so your ldap files are saved outside the container in data volumes.

Be careful, if you remove the container, data volumes will me removed too, except if you have linked this data volume to an other container.

For more information about docker data volume, please refer to :

> [https://docs.docker.com/userguide/dockervolumes/](https://docs.docker.com/userguide/dockervolumes/)

	
### Use an existing ldap database

This can be achieved by mounting host directories as volume. 
Assuming you have a LDAP database on your docker host in the directory `/data/slapd/database`
and the corresponding LDAP config files on your docker host in the directory `/data/slapd/config`
simply mount this directories as a volume to `/var/lib/ldap` and `/etc/ldap/slapd.d`:

	docker run -v /data/slapd/database:/var/lib/ldap \
	-v /data/slapd/config:/etc/ldap/slapd.d
	-d osixia/openldap

You can also use data volume containers. Please refer to :
> [https://docs.docker.com/userguide/dockervolumes/](https://docs.docker.com/userguide/dockervolumes/)

### Using TLS

#### Use autogenerated certificate
By default TLS is enable, a certificate is created for the CN (common name) ldap.example.org. To work properly on your server adjust SERVER_NAME environment variable to match the ldap server CN.

	docker run -e SERVER_NAME=ldap.my-compagny.com -d osixia/openldap

#### Use your own certificate

Add your custom certificate, private key and CA certificate in the directory **image/service/slapd/assets/ssl** adjust filename in **image/env.yml** and rebuild the image ([see manual build](#manual-build)).

Or you can set your custom certificate at run time, by mouting your a directory containing thoses files to **/osixia/slapd/ssl** and adjust there name with the following environment variables :

	docker run -v /path/to/certifates:/osixia/slapd/ssl \
	-e SSL_CRT_FILENAME=my-ldap.crt \
	-e SSL_KEY_FILENAME=my-ldap.key \
	-e SSL_CA_CRT_FILENAME=the-ca.crt \
	-d osixia/openldap
	
#### Disable TLS
Add -e USE_TLS=false to the run command :

	docker run -e USE_TLS=false -d osixia/openldap

## Administrate your ldap server
If you are looking for a simple solution to administrate your ldap server you can take a look at our phpLDAPadmin docker image :
> [osixia/phpldapadmin](https://github.com/osixia/docker-phpLDAPadmin)

## Environment Variables

Environement variables defaults are set in **image/env.yml**. You can modify environment variable values directly in this file and rebuild the image ([see manual build](#manual-build)) or you can override those values at run time with -e argument. See example below.

Required for new ldap server :
- **LDAP_ORGANISATION**: Organisation name. Defaults to `Example Inc.`
- **LDAP_DOMAIN**: Ldap domain. Defaults to `example.org`
- **LDAP_ADMIN_PASSWORD** Admin password. Defaults to `admin`

TLS options :
- **USE_TLS**: Add openldap TLS capabilities. Defaults to `true`
- **SSL_CRT_FILENAME**: Ldap ssl certificate filename. Defaults to `ldap.crt`
- **SSL_KEY_FILENAME**: Ldap ssl certificate private key filename. Defaults to `ldap.key`
- **SSL_CA_CRT_FILENAME**: Ldap ssl CA certificate  filename. Defaults to `ca.crt`
- **SERVER_NAME**: Use by autogenerated certificate: Server CN. Defaults to `ldap.example.org`

### Set environment variables at run time :

Environment variable can be set directly by adding the -e argument in the command line, for example :
	
	docker run -e LDAP_ORGANISATION="My Compagny" -e LDAP_DOMAIN="my-compagny.com" \
	-e LDAP_ADMIN_PASSWORD="JonSn0w" -d osixia/openldap

## Manual build

Clone this project :

	git clone https://github.com/osixia/docker-openldap
	cd docker-openldap

Adapt Makefile, set your image NAME and VERSION, for example :

	NAME = osixia/openldap
	VERSION = 0.10.0
	
	becomes :
	NAME = billy-the-king/openldap
	VERSION = 0.1.0

Build your image :
	
	make build
	
Run your image :

	docker run -d billy-the-king/openldap:0.1.0

## Tests

We use **Bats** (Bash Automated Testing System) to test this image:

> [https://github.com/sstephenson/bats](https://github.com/sstephenson/bats)

Install Bats, and in this project directory run :

	make test

	
