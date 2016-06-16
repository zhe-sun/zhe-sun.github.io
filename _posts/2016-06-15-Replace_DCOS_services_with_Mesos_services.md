---
layout: post
title: Replace DC/OS services with Mesos's
description: "Test for replace DC/OS services with Mesos's"
modified: 2016-06-15T15:27:45-04:00
tags: [Mesos DC/OS]
image:
  feature: abstract-10.jpg
---


**This Is A Test for replacing the libs of DC/OS with the Mesos's.**  


In this test, we can know the correlation between libs clearly. OK, let us have a try, for funny:  

## DCOS ##

1.	Example for mesos-slave


	$ vim dcos-mesos-slave.service  
					
	[Unit]
	Description=Mesos Agent: DCOS Mesos Agent Service   
	[Service]
	Restart=always
	StartLimitInterval=0
	RestartSec=5
	KillMode=control-group
	Delegate=true
	LimitNOFILE=infinity
	EnvironmentFile=/opt/mesosphere/environment
	EnvironmentFile=/opt/mesosphere/etc/mesos-slave-common
	EnvironmentFile=-/var/lib/dcos/mesos-slave-common
	EnvironmentFile=/var/lib/dcos/mesos-resources
	ExecStartPre=/bin/ping -c1 ready.spartan
	ExecStartPre=/bin/ping -c1 leader.mesos
	ExecStart=/opt/mesosphere/packages/mesos--bcd3532be711ab9e0828c963c07a5a0581ca0757/bin/mesos-slave
	EnvironmentFile=-/var/lib/dcos/environment.proxy    
  

2.	In the file, the /opt/mesosphere/etc/mesos-slave-common is very important.

	```
		$ vim /opt/mesosphere/etc/mesos-slave-common
				
		MESOS_MASTER=zk://leader.mesos:2181/mesos
		MESOS_CONTAINERIZERS=docker,mesos
		MESOS_LOG_DIR=/var/log/mesos
		MESOS_MODULES=file:///opt/mesosphere/etc/mesos-slave-modules.json
		MESOS_CONTAINER_LOGGER=org_apache_mesos_LogrotateContainerLogger
		MESOS_ISOLATION=cgroups/cpu,cgroups/mem,posix/disk,com_mesosphere_StatsIsolatorModule
		MESOS_WORK_DIR=/var/lib/mesos/slave
		MESOS_SLAVE_SUBSYSTEMS=cpu,memory
		MESOS_EXECUTOR_ENVIRONMENT_VARIABLES=file:///opt/mesosphere/etc/mesos-executor-environment.json
		MESOS_EXECUTOR_REGISTRATION_TIMEOUT=10mins
		MESOS_CGROUPS_ENABLE_CFS=true
		MESOS_CGROUPS_LIMIT_SWAP=false
		MESOS_DOCKER_REMOVE_DELAY=1hrs
		MESOS_GC_DELAY=2days
		MESOS_HOSTNAME_LOOKUP=false
		GLOG_drop_log_memory=false
		MESOS_HOOKS=com_mesosphere_StatsEnvHook
	```

3. Find the "MESOS_ISOLATION" in <kbd>/opt/mesosphere/packages/dcos-image--75d4ba1419141c3d791b9eaa90f4483503b17777/lib/python3.4/site-packages/gen/dcos-config.yaml</kbd>   
In the dcos-config.yaml:   

	```
	...
	MESOS_ISOLATION={{ mesos_isolation_modules }}
	...   
	```

4. Then we find <kbd>mesos_isolation_modules</kbd> in    
<kbd>/opt/mesosphere/packages/dcos-image--75d4ba1419141c3d791b9eaa90f4483503b17777/lib/python3.4/site-packages/gen/calc.py</kbd>   

	```
	...
	'mesos_isolation_modules': ','.join(__default_isolation_modules + [
	                       __stats_isolator_slave_module_name]),
	...
	```
5. Then in __default-isolation-modules, the content is below:  

	```
	__default_isolation_modules = [
		'cgroups/cpu',
	    'cgroups/mem',
	    'posix/disk',
	]  
	```

6. About <kbd>__stats-isolator-slave-module-name</kbd>:  
	```css
	...
	__stats_isolator_slave_module_name = 'com_mesosphere_StatsIsolatorModule'
	...   
	```

7. And <kbd>--stats-isolator-slave-module-name</kbd> is in <kbd>_stats-slave-module</kbd>:  

	```css
	__stats_slave_module = {
	    'file': '/opt/mesosphere/lib/libstats-slave.so',
	    'modules': [{
	        'name': __stats_isolator_slave_module_name,
	    }, {
	        'name': __stats_hook_slave_module_name,
	        'parameters': [
	            {'key': 'dest_host', 'value': 'metrics.marathon.mesos'},
	            {'key': 'dest_port', 'value': '8125'},
	            {'key': 'dest_refresh_seconds', 'value': '60'},
 				{'key': 'listen_host', 'value': '127.0.0.1'},
				{'key': 'listen_port_mode', 'value': 'ephemeral'},
		        {'key': 'annotation_mode', 'value': 'key_prefix'},
		        {'key': 'chunking', 'value': 'true'},
	            {'key': 'chunk_size_bytes', 'value': '512'},
	        ]
	    }]
	}
	```

8. And where is the <kbd>--stats-slave-module</kbd>?
  
	```
	entry { 
		...
		'mesos_slave_modules_json': calculate_mesos_slave_modules_json(
           	            __default_mesos_slave_modules + [__stats_slave_module]),
		...
		}
	```

9. The file <kbd>--default-mesos-slave-modules</kbd>:  

	```
	__default_mesos_slave_modules = [
		   __logrotate_slave_module,
	]
	```

10. The contents of <kbd>--logrotate-slave-module</kbd>:  

	```ruby
	__logrotate_slave_module = {
			 'file': '/opt/mesosphere/lib/liblogrotate_container_logger.so',
			 'modules': [{
			 'name': __logrotate_slave_module_name,
			 'parameters': [
			  {'key': 'launcher_dir', 'value': '/opt/mesosphere/active/mesos/libexec/mesos/'},
			  {'key': 'max_stdout_size', 'value': '2MB'},
			  {'key': 'logrotate_stdout_options', 'value': 'rotate 9'},
			  {'key': 'max_stderr_size', 'value': '2MB'},
			  {'key': 'logrotate_stderr_options', 'value': 'rotate 9'},
		 	]
		}]
	}
	```

11. Where is the <kbd>mesos_slave-modules_json</kbd>:  <kbd>/opt/mesosphere/packages/dcos-image--75d4ba1419141c3d791b9eaa90f4483503b17777/lib/python3.4/site-packages/gen/dcos-config.yaml</kbd>  

	```
	package {
	...
		- path: /etc/mesos-slave-modules.json
		  content: |
		  {{ mesos_slave_modules_json }}
	...
	}

    ```
		

## Mesos ##

1. ./bin/mesos-slave.sh  

	```
	. /root/repo/mesos/build/bin/mesos-agent-flags.sh
	exec /root/repo/mesos/build/src/mesos-agent "${@}"
	```

>**NOTE:**  
>OK, when replace the libs of DC/OS with the Mesos's, it doesn't work! And when roll back to the default configure of DC/OS, it still does not work.  
>So, I guess, something in libs of DC/OS have changed. I should find another way to enable GPUs in DC/OS.   
>And I know that it is not only the libs needed to modify, but also the interface, the stuffs of 3th-party components, and so on.  
>**BTW, THIS IS A TEST!**