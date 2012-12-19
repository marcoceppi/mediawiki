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

Once bootstrapped, deploy the [MySQL][3] and MediaWiki charm:

    juju deploy mysql
    juju deploy mediawiki

Add a relation between the two. Note: To avoid recieving "ambiguous relation" error, specify the "db" relation:

    juju add-relation mysql mediawiki:db

Expose the MediaWiki service

    juju expose mediawiki

## Additional Options

In addition to the required minimum of MySQL -> MediaWiki; this charm also accepts several other services to provide a more robust service experience.

### Memcached

MediaWiki is designed to cache against memcached to provide a faster and smoother site experience. To add memcached to your mediawiki service first 
deploy memcached:

    juju deploy memcached

then relate it to the mediawiki service

    juju add-relation memcached mediawiki

Memcached is recommended for environments with more than one unit deployed. Otherwise there is very little advantage gained by using memcached since 
MediaWiki will already use whatever byte-code cache is specified in the charm's configuration.

### MySQL Slave

If you're running MySQL with a slave set up you can attach MediaWiki to those slaves directly as MediaWiki (both the application and service) can handle this. To 
do this first set up a slave relation with MySQL (If you've already done so skip to the next set of commands):

    juju deploy mysql mysql-slave
    juju add-relation mysql mysql-slave

Going forward you can scale out MySQL by adding slaves via `juju add-unit mysql-slave`.

Create a relation between the new slave services and MediaWiki:

    juju add-relation mediawiki:slave mysql-slave

# Configuration

MediaWiki charm comes with a handful of settings designed to help streamline and manage your deployment. For convenience if any applicable MediaWiki setting variables are 
associated with the change they'll be listed in parentheses ().

## MediaWiki name ($wgSitename)

This will set the name of the Wiki installation.

    juju set mediawiki name='Juju Wiki!'

## Skin ($wgDefaultSkin)

As the option implies, this sets the default skin for all new users and anonymous users.

    juju set mediawiki skin='monobook'

One limitation is already registered users will have whatever Skin was set as the default applied to their account. This is a [MediaWiki "limitation"][4]. See caveats 
for more information on running Maintenance scripts.

# Caveats

## Maintenance Scripts

From time to time, during routine operation of a MediaWiki installation, maintenace via [the maintenace scripts directory][8] may need to be executed. Depending on the nature of the maintenance operation this might need to occur on one node or all nodes of a MediaWiki deployment.
The following examples outline how to run maintenance on either a single machine or all units in a service.

### Single unit

At any time you can use juju ssh to access any node in your deployment. Juju ssh is a documented feature of Juju, you can learn more [about it on Ask Ubuntu][9].

### All units

In the event you need to run a script on all machines at once you can use the following bash loop (replacing:

    maint_script_to_run=""
    
    for unit in `juju status wordpress | egrep -E "machine: ([0-9])" | tr -d ' ' | cut -d ':' -f2`; do
        juju ssh $unit "php -q /var/www/maintenance/$maint_script_to_run"
    done

# Additional Information

 * Maintainer: Clint Byrum
 * Charm Info: [Mediawiki on Juju Charms](http://jujucharms.com/charms/precise/mediawiki)
 * Bug Reports: [Mediawiki bugs on LaunchPad](https://bugs.launchpad.net/charm/precise/+source/mediawiki)

[1]: https://juju.ubuntu.com/docs/getting-started.html
[2]: https://juju.ubuntu.com/docs/getting-started.html#installation
[3]: http://jujucharms.com/charms/precise/mysql
[4]: http://www.mediawiki.org/wiki/Manual:$wgDefaultSkin
[5]: http://packages.ubuntu.com/precise/mediawiki
[6]: http://www.mediawiki.org/wiki/Download_from_Git
[7]: https://integration.mediawiki.org/nightly/mediawiki/core/?C=M;O=D
[8]: http://www.mediawiki.org/wiki/Manual:Maintenance_scripts
[9]: http://askubuntu.com/questions/152428/how-to-ssh-into-local-juju-instance
