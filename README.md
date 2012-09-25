Icinga-scripts
==============

Customized scripts to use with the icinga (1.6.x) monitoring tool on CentOS 6 machines.

I created some customized checks using the icinga nrpe daemon. I loaded them up in this repository to share them with the world.

Feel free to share your remarks!

Configuration icinga
====================

Server side:
------------

I added the commands to the file '/etc/icinga/objects/commands.cfg' like this:

	define command {
		command_name    check_NAME	
		command_line    $USER1$/check_NAME
	}	

To configure the specified service I added this config to a node in '/etc/icinga/objects/services':

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

On the client we need to add the script into for example the default directory '/usr/lib64/nagios/plugins' make sure it has executable permissions! (chmod +x)

Also the nrpe configuration, '/etc/nagios/nrpe.cfg' needs to be adopted:

	"command[check_NAME]=/usr/lib64/nagios/plugins/check_NAME"

and restart the nrpe daemon (/etc/init.d/nrpe restart)

After a few minutes the check should appear into your icinga front-end.

Jenkins
=======

(I wrote this first check to see how icinga needs to be configured to manage customized scripts.)

This check will use the output of the command '/etc/init.d/jenkins status' and will count the updates related to jenkins 'yum check-update | grep jenkins | wc -l' to throw some messages to the icinga server:

	OK status:
	OK: jenkins (pid  XXXX) is running / OK: No updates available

	Warning status:
	WARNING: jenkins (pid  XXXX) is stopped / WARNING: X updates available

	Critical status:
	CRITICAL: jenkins is not running / OK or WARNING state about updates overridden by the CRITICAL state of the service
