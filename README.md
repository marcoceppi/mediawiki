# Overview

MediaWiki is a free wiki software application. Developed by the Wikimedia Foundation and others, it is used to run all of the projects
hosted by the Foundation, including Wikipedia, Wiktionary and Commons. Numerous other wikis around the world also use it to power their 
websites. It is written in the PHP programming language and uses a backend database.

This charm will deploy MediaWiki in to the cloud while applying best practices for running scale-out infrastructures, php based applications 
and MediaWiki.

# Deployment

This charm is available in the Juju Charm Store along with hundreds of others. To deploy this charm you will need: [a cloud environment][1], a working [Juju][2] 
installation, and an already bootstrapped environment.

## Required

Once bootstrapped, deploy the MySQL and MediaWiki charm:

    juju deploy mysql
    juju deploy mediawiki

Add a relation between the two. Note: To avoid recieving "ambiguous relation" error, specify the "db" relation:

    juju add-relation mysql mediawiki:db

Expose the MediaWiki service

    juju expose mediawiki

## Optional Additions

In addition to the required minimum of MySQL -> MediaWiki; this charm also accepts several other services to provide a more robust service experience.

### Memcached

MediaWiki is designed to cache against memcached to provide a faster and smoother site experience. To add memcached to your mediawiki service first 
deploy memcached

    juju deploy memcached

then relate it to the mediawiki service

    juju add-relation memcached

Memcached is recommended for environments with more than one unit deployed. Otherwise there is very little advantage gained by using memcached since 
MediaWiki will already use whatever byte-code cache is specified in the charm's configuration.

### MySQL Slave

If you're running MySQL with a slave set up you can attach MediaWiki to those slaves directly as MediaWiki (both the application and service) can handle this. To 
do so execute the following:

    juju add-relation mediawiki mysql-slave-service:slave


