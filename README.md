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

## Optional Additions

In addition to the required minimum of MySQL -> MediaWiki; this charm also accepts several other services to provide a more robust service experience.

### Memcached

MediaWiki is designed to cache against memcached to provide a faster and smoother site experience. To add memcached to your mediawiki service first 
deploy memcached:

    juju deploy memcached

then relate it to the mediawiki service

    juju add-relation memcached

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

## Version

In the event you wish to run a version of MediaWiki _other_ than what is provided by the [Ubuntu Archives][5] you can change the version option. Version configuration option has 
several "magic" values to provide easy short cuts.

### distro

This is the default version and will install mediawiki from the [Ubuntu Archives][5]. This is typically the safest option.

    juju set mediawiki version=distro

### latest

Latest installs the latest stable release from upstream. For backwards compatibility the option "upstream" is an alias of this. For this installation, MediaWiki will be installed from 
the [upstream git repository][6] and the last stable tag checked out. This, coupled with the `auto-update` configuration option is a good way to insure you have the latest stable always running.

    juju set mediawiki version=latest

### nightly

Nightly will install MediaWiki from the [Nightlies repo][7] and is not recommended for production. Instead this is designed for testing purposes. When coupled with the `auto-update` configuration option a 
new version will be installed on a nightly basis.

    juju set mediawiki version=nightly

### trunk

This will pull whatever is the latest from the [MediaWiki git repository's][6] trunk. Not recommended for production. When paired with `auto-update` this will refresh the trunk approx every 30 mins.

    juju set mediawiki version=trunk

### X.Y.Z

In addition to each of the "magic values", you can set a specific release of MediaWiki or a Wildcard match for a version number. For example, if you wish to use just the 1.20 release of MediaWiki set the 
value of `version` to "1.20". If you want all minor releases under 1.20 set the value to `1.20.X` (Both `X` and `*` are valid wildcard flags).

    juju set mediawiki version=1.19.2

When paired with `auto-update` and a wildcard version, this charm will check for updates twice a day.

    juju set mediawiki version="1.20.X"

## auto-update

# Caveats

## Maintenance Scripts



# Additional Information



[1]: 
[2]: https://juju.ubuntu.com/docs/getting-started.html#installation
[3]: http://jujucharms.com/charms/precise/mysql
[4]: http://www.mediawiki.org/wiki/Manual:$wgDefaultSkin
[5]: http://packages.ubuntu.com/precise/mediawiki
[6]: http://www.mediawiki.org/wiki/Download_from_Git
[7]: https://integration.mediawiki.org/nightly/mediawiki/core/?C=M;O=D

