Icinga-scripts
==============

Customized scripts to use with the icinga (1.6.x) monitoring tool on CentOS 6 machines. I implemented multi OS support using a script from https://github.com/coto/server-easy-install/blob/master/lib/core.sh for the bacula updates.

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
	OK: jenkins (pid  XXXX) is running / OK: No updates available / OK: Config file /var/lib/jenkins/config.xml is present 

	Warning status:
	WARNING: jenkins (pid  XXXX) is stopped / WARNING: X updates available / OK: Config file /var/lib/jenkins/config.xml is present

	Critical status:
	CRITICAL: jenkins is not running / OK or WARNING state about updates overridden by the CRITICAL state of the service / Critical: Config file /var/lib/jenkins/config.xml does not exists

Example output Status Information in icinga:

        OK: jenkins (pid 29397) is running... / OK: No updates available / OK: Config file /var/lib/jenkins/config.xml is present / 

Optional output when using the script will be added to this previous line:

        OK: jobs count: 95 - jobs=95:: passed=95 failed=0:100:100 disabled=0 running=0

Bacula
======

Make sure the bconsole configuration file has got the proper permissions (chmod 644 /etc/bacula/bconsole.conf). Else the nrpe daemon could not read the output!

This check will use the output of

*  the command '/etc/init.d/bacula-fd status'
*  counts the updates related to jenkins 'yum check-update | grep bacula | wc -l'
*  checks if the config file '/etc/bacula/bacula-fd.conf' is still present
*  ( if you comment out line 31, adapt it to the right bacula client name, delete the underlaying declaration of $BACKUP and uncomment line 89 and delete line 90 the script from Michael Wyraz will be used to get the time since the last backup has been taken. By default, exceeding 24hrs the warning state is initialized, more than 48hrs it will become critical - http://exchange.nagios.org/directory/Plugins/Backup-and-Recovery/Bacula/check_bacula_lastbackup-2Epl/details)

to throw some messages to the icinga server:

        OK status:
        OK: bacula-fd (pid  XXXX) is running / OK: No updates available / OK: Config file /etc/bacula/bacula-fd.conf is present

        Warning status:
        WARNING: bacula-fd (pid  XXXX) is stopped / WARNING: X updates available / OK: Config file /etc/bacula/bacula-fd.conf is present

        Critical status:
        CRITICAL: bacula-fd is not running / OK or WARNING state about updates overridden by the CRITICAL state of the service / Critical: Config file /etc/bacula/bacula-fd.conf

Example output Status Information in icinga:

Without the optional script:

        OK: bacula-fd (pid 30335) is running... / OK: No updates available / OK: Config file /etc/bacula/bacula-fd.conf is present

Optional output when using the script will be added to this previous line:

        OK: Last backup for asturias was 14:52 hours ago.

YUM
===

!Until now I did not managed to make it work through the NRPE daemon yet, probably a permission issue!

This check will read out the time when the latest update was made on the remote host using the output of

* yum history | grep -n "U" 

and throw this message to the icinga server:

        OK status:
        OK: Last update performed on YYYY-MM-DD at HH:MM  
       
        Warning status:
        WARNING: Last update performed a long time ago 

Example output Status Information in icinga:

        OK: Last update performed on 2012-09-26 at 14:29
