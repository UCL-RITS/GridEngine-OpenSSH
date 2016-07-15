# OpenSSH-GridEngine integration

##Existing integrations

On a typical grid engine base cluster grid-engine and ssh interact in two ways:

* When qrsh uses ssh as an rsh_command to launch tasks on a node.

* When a parallel library invokes ssh and the sysadmin provides a wrapper script with that name that translates options and invokes qrsh in order to tightly integrate jobs.

##Problems with existing integrations

This second use can  cause two problems:

* A user of the cluster tries to use ssh to connect to a machine outside the cluster and accidentally  uses the wrapper script instead  which typically will not work.

* The wrapper script typically only implements a subset of options so new parallel libraries require modifying the wrapper or writing a new one.  One can end up with multiple PEs just to support different wrappers with different options.

##UCL Solution 

At UCL we use a set of scripts that allow us to use the real ssh binary provided by OpenSSH as a wrapper around qrsh while still supporting parallel libaries that invoke qrsh directly.  Obviously the real ssh implements the full set of ssh command line switches.  By using host specific config the ssh binary can be tightly integrated with grid engine while still providing normal access to hosts outside the cluster.  The scripts we use are provided as 'bin/rshd' 'bin/proxycommand' 'bin/sshorig' and 'bin/rshcommand'.  

##Known Limitations 

* The integration scripts invoke each other based on the assumption that they are all in the same directory.

* The integration is not compatible with ssh connection sharing ('ControlMaster').

* The integration relies on setting an ssh 'ProxyCommand' for hosts within the cluster.  Users can disable this or set their own which will break the integration.

##HOWTO

If you are using the traditional method of ssh tight integartion then following these instructions in order should 
allow a live upgrade but this is untested and whether it is wise to attempt this live is up to you.

1.Make sure you have passwordless ssh working between your execution hosts and from submit hosts to execution hosts.

2.Install the integration scripts somewhere they are accessible to your execution hosts (and optionally your login hosts).  Check that the paths in the scripts are correct for your system.  In particular make sure that the HIGHHEELS variable in the proxycommand script points to a version of netcat(or similar) that takes the host and port to connect to as its first and second arguments respectively.  

3.If you want users who ssh from login nodes to execution nodes to do so under gridengine control (ie as a job) then create
a file with options to be passed to qrsh and place it in the appropriate location (../etc/ssh_request) relative to the location of the proxycommand script.

4.Use qconf to alter the 'rsh_daemon' setting in the grid engine config('qconf  -Mconf' OR 'qconf -mconf') to point to the provided rshd script .  You can optionally specify an alternative file to use as the sshd_config as an argument to the script.

5.Use qconf to alter the 'rsh_command' setting in the grid engine config('qconf -Mconf' OR 'qconf -mconf' again) to point to the provided rshcommand script.

6.On your execution hosts(and optionally your submit nodes) in the '/etc/ssh/ssh_config' file add a Host section covering the execution hosts (by short hostname,fqdn and IP address) that includes a Proxycommand that points to the provided proxycommand script.  If something (eg: your node installer) automatically appends additional ssh config ito the file you may want to follow it with a Host * section to ensure that additional config applies to all hosts not just your execution hosts.  Something like this:
'''
Host node-????-??? node-????-???.data.legion.ucl.ac.uk 10.143.*
ProxyCommand /opt/geassist/bin/proxycommand %h %p
Host *
'''

7.Enjoy.












