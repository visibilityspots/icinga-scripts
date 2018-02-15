Icinga-scripts
==============

Customized scripts to use with the icinga (1.6.x) monitoring tool on CentOS 6 machines. I implemented multi OS support using a script from https://github.com/coto/server-easy-install/blob/master/lib/core.sh for the bacula updates. And check_bacula-lastbackup from Michael Wyraz (http://exchange.nagios.org/directory/Plugins/Backup-and-Recovery/Bacula/check_bacula_lastbackup-2Epl/details) which checks the last bacula backup againts a specific period of hours.

Those scripts should be stored into the nagios plugin (/usr/lib64/nagios/plugins) folder of the hosts you want to monitor, they require the utils.pm library from the icinga instance. Using the NRPE daemon they will be executed.

Feel free to share your remarks!

Configuration icinga
====================

Server side:
------------

Add the commands to the file '/etc/icinga/objects/commands.cfg' like this:

	define command {
		command_name    check_NAME
		command_line    $USER1$/check_NAME
	}

To configure the specified service this config is needed for a node in '/etc/icinga/objects/services':

	define service {
        	service_description            DESCRIPTION
	        check_command                  check_nrpe_command!check_NAME
        	use                            generic-service
	        notification_period            24x7
	        host_name                      HOSTNAME.OF.SERVER
	}

And restart the icinga service (/etc/init.d/icinga restart)

Client side:
------------

On the client add the script into for example the default directory '/usr/lib64/nagios/plugins' make sure it has executable permissions! (chmod +x)

Also the nrpe configuration, '/etc/nagios/nrpe.cfg' needs to be adopted:

	"command[check_NAME]=/usr/lib64/nagios/plugins/check_NAME"

and restart the nrpe daemon (/etc/init.d/nrpe restart)

After a few minutes the check should appear into your icinga front-end.

Jenkins
=======

Make sure the jenkins configuration file has got the proper permissions (chmod 644 /etc/sysconfig/jenkins). Else the nrpe daemon could not read the output!

This check will use the output of

*  the command '/etc/init.d/jenkins status'
*  counts the updates related to jenkins 'yum check-update | grep jenkins | wc -l'
*  checks if the config file '/var/lib/jenkins/config.xml' is still present
*  ( if you comment out lines 18 and 58 the script from Eric Blanchard will be used to get total number of jobs of the jenkins instance -  https://github.com/Ericbla/check_jenkins/blob/master/check_jenkins.pl )

to throw some messages to the icinga server:

	OK status:
	OK: jenkins (pid  XXXX) is running... / OK: No updates available / OK: Config file /var/lib/jenkins/config.xml is present

	Warning status:
	WARNING: jenkins (pid  XXXX) is stopped / WARNING: X updates available / OK: Config file /var/lib/jenkins/config.xml is present

	Critical status:
	CRITICAL: jenkins is not running / OK or WARNING state about updates overridden by the CRITICAL state of the service / Critical: Config file /var/lib/jenkins/config.xml does not exists

Optional output when using the script will be added to this previous line:

        OK: jobs count: XX - jobs=XX:: passed=XX failed=XX:100:100 disabled=0 running=0

Bacula
======

Make sure the bconsole configuration file has got the proper permissions (chmod 644 /etc/bacula/bconsole.conf). Else the nrpe daemon could not read the output!

This check will use the output of

*  the command '/etc/init.d/bacula-fd status'
*  counts the updates related to jenkins 'yum check-update | grep bacula | wc -l'
*  checks if the config file '/etc/bacula/bacula-fd.conf' is still present
*  ( if you comment out line 31, adapt it to the right bacula client name, delete the underlying declaration of $BACKUP and uncomment line 89 and delete line 90 the script from Michael Wyraz will be used to get the time since the last backup has been taken. By default, exceeding 24hrs the warning state is initialized, more than 48hrs it will become critical - http://exchange.nagios.org/directory/Plugins/Backup-and-Recovery/Bacula/check_bacula_lastbackup-2Epl/details)

to throw some messages to the icinga server:

        OK status:
        OK: bacula-fd (pid  XXXX) is running / OK: No updates available / OK: Config file /etc/bacula/bacula-fd.conf is present

        Warning status:
        WARNING: bacula-fd (pid  XXXX) is stopped / WARNING: X updates available / OK: Config file /etc/bacula/bacula-fd.conf is present

        Critical status:
        CRITICAL: bacula-fd is not running / OK or WARNING state about updates overridden by the CRITICAL state of the service / Critical: Config file /etc/bacula/bacula-fd.conf

Optional output when using the script will be added to this previous line:

        OK: Last backup for XXXXXX was XX:XX hours ago.

User
====

This check will look for the users who are logged into the server by using the output of

* w

and throw this message to the icinga server depending on the idle time (if more than 1 hour idle* then warning):

        OK status:
        OK: Active user(s): XXXXXX logged in at XX:XX or OK: No users logged in

        Warning status:
        WARNING: Inactive user(s): XXXXXX passive for XX:XXm

* Idle time is the time since the last shell activity has been made

Memory
======

This check will calculates the available and used memory percentages of the monitored host using the output of;

```bash
$ vmstat -s
$ cat /proc/meminfo | grep MemAvailable
```

```bash
available
        Estimation  of how much memory is available for starting new applications, without swapping. Unlike the data provided by the cache or free fields, this field takes into account page cache and also that not all reclaimable memory slabs will be reclaimed
        due to items being in use (MemAvailable in /proc/meminfo, available on kernels 3.14, emulated on kernels 2.6.27+, otherwise the same as free)
```

and throw this message to the icinga server depending on the calculated percentage (if less than 15 % available: warning state if less than 10 % available memory: critical);

        OK status:
        OK: XX% of memory available, XX% is used / OK: XX% of swap used, XX% is free

        Warning status:
        WARNING: only XX% of memory available, only XX% is used / WARNING: XX% of swap used, only XX% is free

        Critical status:
        CRITICAL: only XX% of memory available, XX% is used / CRITICAL: XX% of swap used, XX% is free

INTERNET
========

This check will see if there is a connection to the outside internet using the output of the wget command

* wget -T 5 -t 5 --delete-after google.com

and throw this message to the icinga server depending on the state of the webpage got from the wget output:

        OK status:
        OK: The outgoing connection to $URL is reachable

        Warning status:
        WARNING: The page $URL is not found

        Critical status:
        CRITICAL: The outgoing connection to $URL isn't reachable

Solr
====

To monitor a solr instance you could use the check_http and monitor the port the solr instance is running on. It's an easy on but has the disadvantage that you don't know if the instance is running fine. You only now if the instance is running.

So I wrote a custom script, this script is based on the PingRequestHandler (http://lucene.apache.org/solr/4_1_0/solr-core/org/apache/solr/handler/PingRequestHandler.html)

	OK status:
	OK: solr health check

	Critical status:
	CRITICAL: solr health check is failing
Yum
===

!Until now I did not managed to make it work through the NRPE daemon yet, probably a permission issue with the files in /etc/yum.repos.d/!

The goal is to make it work together with the yum script on https://code.google.com/p/check-yum/ so the desired output of the time since last update + number of available updates will be displayed.

This check will read out the time when the latest update was made on the remote host using the output of

* yum history | grep -n "U"

and throw this message to the icinga server:

        OK status:
        OK: Last update performed on YYYY-MM-DD at HH:MM

        Warning status:
        WARNING: Last update performed a long time ago
