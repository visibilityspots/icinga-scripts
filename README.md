Icinga-scripts
==============

Customized scripts to use with the icinga (1.6.x) monitoring tool on CentOS 6 machines.

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

Make sure the jenkins service has got the proper permissions (chmod 644 /etc/sysconfig/jenkins). Else the nrpe daemon could not read the output!

This check will use the output of 

*  the command '/etc/init.d/jenkins status'
*  counts the updates related to jenkins 'yum check-update | grep jenkins | wc -l'
*  checks if the config file '/var/lib/jenkins/config.xml' is still present
(*  if you comment out lines 18 & 58 the script from Eric Blanchard will be used to get total number of jobs of the jenkins instance -  https://github.com/Ericbla/check_jenkins/blob/master/check_jenkins.pl )

to throw some messages to the icinga server:

	OK status:
	OK: jenkins (pid  XXXX) is running / OK: No updates available / OK: Config file /var/lib/jenkins/config.xml is present 

	Warning status:
	WARNING: jenkins (pid  XXXX) is stopped / WARNING: X updates available / OK: Config file /var/lib/jenkins/config.xml is present

	Critical status:
	CRITICAL: jenkins is not running / OK or WARNING state about updates overridden by the CRITICAL state of the service / Critical: Config file /var/lib/jenkins/config.xml does not exists

Example output Status Information in icinga:

jenkins (pid 29397) is running... / OK: No updates available / OK: Config file /var/lib/jenkins/config.xml is present / OK: jobs count: 95 - jobs=95:: passed=95 failed=0:100:100 disabled=0 running=0 
