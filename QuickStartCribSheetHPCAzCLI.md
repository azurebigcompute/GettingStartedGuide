# QUICKSTART Crib Sheet: Deploying an HPC Cluster in Azure 

There are three primary methods to deploy HPC clusters in Azure. 

1) Via portal.azure.com graphical interface, building the cluster manually. 
2) ARM (Azure Resource Manager) templates which define the cluster in JSON code. 
3) Azure CLI commands and scripts. 

## Deploy via ARM templates

The advantage of ARM templates is that they contain all the code to deploy the Azure infrastructure, and also post-installation scripts to customize the image dynamically. The templates can be customized to let the user change parameters (such as number or type of compute nodes) at the time of deployment. 
Example ARM templates for building clusters to run StarCCM & Ansys fluent jobs: 
https://github.com/tanewill/5clickTemplates/tree/master/RawStarCCMCluster
https://github.com/tanewill/5clickTemplates/tree/master/RawANSYSCluster

If you want to use ARM templates it suggested that you use a github account, and clone the template you like into your own github repository, where you can customize it according to your needs and preferences. 

## Deploy by Azure CLI using a customized marketplace image

Another method available, and often preferred by Linux administrators is to deploy from the azure command line interface (Azure CLI 2.0). This allows you to do all the same things as an ARM template using Linux CLI or scripts. 

Deploying from a pre-customized image rather than a stock marketplace image has the advantage that less needs to be done with installation scripts at the time of deployment, which can make the deployment faster (no rpm's or other packages to install, or configuration files to change etc. during deployment). 

A good overall explanation of manual RDMA setup here, but be aware that this is azure CLI 1.0 not 2.0 (2.0 command is "az" rather than "azure"). 
Also this uses "availability sets", whereas these days we recommend to use "scale sets" to deploy HPC clusters
(availability sets & scale sets ensure InfiniBand fabric connectivity between the nodes via switch affinity). 
https://docs.microsoft.com/en-us/azure/virtual-machines/linux/classic/rdma-cluster

### Install azure CLI 2.0

Note, you can do this natively on linux/macox/windows, but you can also do this using the bash for windows: https://msdn.microsoft.com/en-us/commandline/wsl/about. 

Azure CLI 2.0 Installation instructions: 
https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
See example scripts here:
https://docs.microsoft.com/en-us/azure/virtual-machines/linux/cli-samples?toc=%2fcli%2fazure%2ftoc.json&bc=%2fcli%2fazure%2fbreadcrumb%2ftoc.json

### Find the HPC Image closest to your need

The -HPC images have the Microsoft RDMA drivers & Intel MPI built in. If there is no HPC image, you will have to add this part manually.
```
$ az vm image list --publisher OpenLogic --all | grep HPC
CentOS-HPC  OpenLogic    6.5    OpenLogic:CentOS-HPC:6.5:6.5.20160408  6.5.20160408
CentOS-HPC  OpenLogic    7.1    OpenLogic:CentOS-HPC:7.1:7.1.20160408  7.1.20160408
```
### Decide which Hardware SKU you want to use

You can read the details about the different VM types in the Azure documentation with details onthe underlying <a href="https://docs.microsoft.com/en-us/azure/virtual-machines/linux/sizes">hardware types</a> and <a href="https://azure.microsoft.com/en-us/pricing/details/virtual-machines/linux/">VM pricing per location</a>. 

Note: The supported locations are 'eastus, eastus2, westus, centralus, northcentralus, southcentralus, northeurope, westeurope, eastasia, southeastasia, japaneast, japanwest, australiaeast, australiasoutheast, brazilsouth, southindia, centralindia, westindia, canadacentral, canadaeast, westus2, westcentralus, uksouth, ukwest, koreacentral, koreasouth'.

To obtain an exhaustive list of up-to-date hardware SKU's available at a given location, you can run this command: 
```
az vm list-sizes --location eastus
```
Or formatted ready for insertion into a .json template:
````
az vm list-sizes --location eastus | tail -n +3 | awk '{printf("\"%s\",\n",$3)}'
````

### Create a VM with the image you chose

(follow the VM create example script: https://docs.microsoft.com/en-us/azure/virtual-machines/scripts/virtual-machines-linux-cli-sample-create-vm-quick-create?toc=%2fcli%2fazure%2ftoc.json). 

Create a resource group: 
```
$ az group create --name Gothenburg --location westeurope
Location    Name
----------  ----------
westeurope  Gothenburg
```

Create the VM: 
```
$ az vm create --name Golden01 --resource-group Gothenburg --location westeurope --image OpenLogic:CentOS-HPC:6.5:6.5.20160408 --size Standard_H16r --storage-sku Standard_LRS --generate-ssh-keys

Location    MacAddress         PowerState    PrivateIpAddress    PublicIpAddress    ResourceGroup
----------  -----------------  ------------  ------------------  -----------------  ---------------
westeurope  00-0D-3A-28-40-E8  VM running    10.0.0.4            52.233.152.35      Gothenburg
```


### Customize the Image

Login to the running Golden01 image, customize it how you like it with appropriate rpms and NFS mounts to your software and models directory etc. 
```
$ ssh 52.233.152.35 -lmk
The authenticity of host '52.233.152.35 (52.233.152.35)' can't be established.
RSA key fingerprint is ab:36:c6:ec:fe:89:c5:f0:b4:be:67:75:5f:71:e3:55.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '52.233.152.35' (RSA) to the list of known hosts.
Enter passphrase for key '/home/mk/.ssh/id_rsa':
[mk@Golden01 ~]$ uname -a
Linux Golden01 2.6.32-431.29.2.el6.x86_64 #1 SMP Tue Sep 9 21:36:05 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
…
```

### Generalize the Image & Prepare for Deployment

Follow this guide: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/capture-image

```
[mk@Golden01 ~]$ sudo waagent -deprovision+user -force
WARNING! The waagent service will be stopped.
WARNING! All SSH host key pairs will be deleted.
WARNING! Cached DHCP leases will be deleted.
WARNING! Nameserver configuration in /etc/resolv.conf will be deleted.
WARNING! root password will be disabled. You will not be able to login as root.
WARNING! mk account and entire home directory will be deleted.
[mk@Golden01 ~]$
[mk@Golden01 ~]$ exit
logout
Shared connection to 52.233.152.35 closed.
```
```
$ az vm deallocate --resource-group Gothenburg --name Golden01
$ az vm generalize --resource-group Gothenburg --name Golden01
$ az image create --resource-group Gothenburg --name GoldenImage01 --source Golden01
Location    Name           ProvisioningState    ResourceGroup
----------  -------------  -------------------  ---------------
westeurope  GoldenImage01  Succeeded            Gothenburg
```

### Deploy the Scale-Set from the Image

https://docs.microsoft.com/en-us/cli/azure/vmss
```
$ az vmss create --name Sweden --resource-group Gothenburg --image GoldenImage01 --vm-sku Standard_H16r --storage-sku Standard_LRS --instance-count 4 --generate-ssh-keys
$ az vmss list-instances --resource-group Gothenburg --name Sweden
  InstanceId  LatestModelApplied    Location    Name      ProvisioningState    ResourceGroup    VmId
------------  --------------------  ----------  --------  -------------------  ---------------  ------------------------------------
           0  True                  westeurope  Sweden_0  Succeeded            GOTHENBURG       8b28bf07-506f-42c1-a596-f58695dc6ad4
           1  True                  westeurope  Sweden_1  Succeeded            GOTHENBURG       256fe62e-4420-4859-b945-16acabd8c580
           2  True                  westeurope  Sweden_2  Succeeded            GOTHENBURG       88811818-837e-4292-abba-a9121ec485c5
           3  True                  westeurope  Sweden_3  Succeeded            GOTHENBURG       1e61b260-5df3-47b3-bd51-2d6dd7a09f9d

$ az vmss list-instance-connection-info  --name Sweden --resource-group Gothenburg
Result
--------------------
52.166.127.169:50000
52.166.127.169:50001
52.166.127.169:50002
52.166.127.169:50003
```
```
$ ssh 52.166.127.169 -lmk -p 50000
The authenticity of host '[52.166.127.169]:50000 ([52.166.127.169]:50000)' can't be established.
RSA key fingerprint is d4:db:78:9f:6c:f4:6a:1c:57:c7:5d:c7:eb:4c:18:a7.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[52.166.127.169]:50000' (RSA) to the list of known hosts.
Enter passphrase for key '/home/mk/.ssh/id_rsa':
[mk@swedept9l000000 ~]$
```

### Check RDMA Pre-Requisites 

For RDMA to work you *must* fulfill *all* of these criteria:
	1. Use a -HPC image, or have the Microsoft RDMA drivers present. 
	2. The VM's must be deployed to a physical SKU that has RDMA capability (H16r, H16mr, A8, A9, or NC24r). 
	3. The VM's must be deployed into the same availability set or, better, the same scale set. 
	4. Passwordless SSH must be setup between the nodes under the account you are using. 

#### Ensure Cross-Node Password-Free SSH is configured

Step 4 is critical, and the source of some practical difficulties. To make this happen use one of the following methods:
	1. Install the keys manually into the relevant user home directory in your "Golden Image" before you deploy, and don't remove the user when you deprovision. 
	2. Create an NFS share from the head node (or elsewhere) and mount the home directory into each image at boot time. 
	3. Run a script to create and copy the keys & configuration. There is an example script linked from this page: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/classic/rdma-cluster. 
	4. Automate 2 or 3 in an ARM template. The ARM templates at the start of this HOWTO guide illustrate how to do this. 
	

#### Create env vars: 
```
[mk@swedept9l000000 ~]$ cat ~/.bashrc | grep MPI
export INTELMPI_ROOT=/opt/intel/impi/5.1.3.181
export I_MPI_FABRICS=shm:dapl
export I_MPI_DAPL_PROVIDER=ofa-v2-ib0
export I_MPI_ROOT=/opt/intel/compilers_and_libraries_2016.2.181/linux/mpi

mk@swedept9l000000 ~]$ cat pingpong.sh
#!/bin/bash
source /opt/intel/impi/5.1.3.181/bin64/mpivars.sh
mpirun -env I_MPI_FABRICS=dapl -env I_MPI_DAPL_PROVIDER=ofa-v2-ib0 -env I_MPI_DYNAMIC_CONNECTION=0 IMB-MPI1 pingpong
```
If your pingpong test is failing, return to the section "Check RDMA Pre-Requisites" and check again. 

Once you are able to run the infiniband tests successfully across all nodes, you are ready to run your StarCCM, Fluent or other MPI jobs on the cluster. 
