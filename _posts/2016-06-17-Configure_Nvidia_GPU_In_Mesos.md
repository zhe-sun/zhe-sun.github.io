---
layout: post
title: Configure and Enable NVIDIA GPU in Mesos
image:
  feature: abstract-4.jpg
  credit: Sunzhe
comments: true
modified: 2016-06-17

---

## Enable NVDIA GPU In Mesos ##

1. The configure flags can be used to enable Nvidia GPU support, as well as specify the installation directories of the nvml header and library files if not already installed in standard include/library paths on the system.  
They will also be used to conditionally build support for Nvidia GPUs into Mesos.  

2. In the initial GPU support we will not do auto-discovery of GPUs on an agent. As such, an operator will need to specify a flag on the command line, listing all of the GPUs available on the system.  


### Configure NVDIA GPU ###

I ran bootstrap to generate configure.  
I then ran:  
mkdir build; cd build  

```bash
../configure --enable-nvidia-gpu-support
../configure --enable-nvidia-gpu-support --with-nvml-include=<relative_path_to_headers>
../configure --enable-nvidia-gpu-support --with-nvml-include=<path_to_headers>
../configure --enable-nvidia-gpu-support --with-nvml-include=<bogus_path>
../configure --enable-nvidia-gpu-support --with-nvml-include=<relative_path_to_lib>
../configure --enable-nvidia-gpu-support --with-nvml-lib=<path_to_lib>
../configure --enable-nvidia-gpu-support --with-nvml-lib=<bogus_path>
../configure --enable-nvidia-gpu-support --with-nvml-include=<bogus_path> --with-nvml-lib=<path_to_lib>
../configure --enable-nvidia-gpu-support --with-nvml-include=<path_to_headers> --with-nvml-lib=<bogus_path>
../configure --enable-nvidia-gpu-support --with-nvml-include=<bogus_path> --with-nvml-lib=<bogus_path>
../configure --enable-nvidia-gpu-support --with-nvml-include=<path_to_headers> --with-nvml-lib=<path_to_lib>
```

And verified the proper errors / successes in each case (only the last one is a success).  
The exact command I ran in the success case for my configuration was:  

```bash
../configure --enable-nvidia-gpu-support --with-nvml-include=/opt/nvidia-gdk/usr/include --with-nvml-lib=/opt/nvidia-gdk/usr/src/gdk/nvml/lib
```
The link: https://issues.apache.org/jira/browse/MESOS-4861  


### Start Agent with Enabling NVIDIA GPU ###

```bash
./bin/mesos-slave.sh --master=127.0.0.1:5050 --resources="gpus:string"
Failed to determine slave resources: Bad type for resource gpus value string type TEXT
	
./bin/mesos-slave.sh --master=127.0.0.1:5050 --resources="gpus:4.9"
Failed to determine slave resources: The gpus resource must specified as an unsigned integer
	
./bin/mesos-slave.sh --master=127.0.0.1:5050 --resources="gpus:4.0"
Failed to determine slave resources: When specifying the gpus resource, you must also specify a list of GPUs via the --nvidia_gpu_devices flag
	
./bin/mesos-slave.sh --master=127.0.0.1:5050 --resources="gpus:4.0" --nvidia_gpu_devices=1,2,3
Failed to determine slave resources: The number of GPUs passed in the --nvidia_gpu_devices flag must match the number of GPUs specified in the gpus resource
	
./bin/mesos-slave.sh --master=127.0.0.1:5050 --resources="gpus:4.0" --nvidia_gpu_devices=1,2,3,4
	SUCCESS
```

> **NOTE:** I didn't set the --isolation flag here, so the agent actually started up correctly (i.e. it didn't error out saying that the Nvidia GPU isolator is currently unsupported).  This is the correct behaviour, and it properly exercises the code added in this patch.  
> 

The link: https://issues.apache.org/jira/browse/MESOS-4865  

### Added flag to specify available Nvidia GPUs on an agent's command line ###

Testing Done  

```bash
./bin/mesos-slave.sh --master=127.0.0.1:5050 --nvidia_gpu_devices=1,2,string
Failed to load flag 'nvidia_gpu_devices': Failed to load value '1,2,string': Expecting all elements to be unsigned integers

./bin/mesos-slave.sh --master=127.0.0.1:5050 --nvidia_gpu_devices=1,2.0,3 
Failed to load flag 'nvidia_gpu_devices': Failed to load value '1,2,3.0': Expecting all elements to be unsigned integers

./bin/mesos-slave.sh --master=127.0.0.1:5050 --nvidia_gpu_devices=1,2,3
SUCCESS
```

The link: https://issues.apache.org/jira/browse/MESOS-4864  

