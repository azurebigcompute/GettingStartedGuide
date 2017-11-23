# Step 1: Get Tooled Up for Azure Big Compute #

Before you begin to work with Azure Big Compute & HPC, we recommend that you take a few minutes to download and install some relevant Azure client side tools that will make working with Azure a lot faster and easier (and more fun)! These utilities can either be installed on your laptop, or you can provision a  linux or windows VM in Azure, and install them there. 

Most of these tools are universal and run on Linux, Windows or MacOS. 

A good option for Linux folks is to install the Linux tools into the Ubuntu Bash shell on Windows 10. There is a quick HOWTO on enabling bash for windows <a href="https://www.windowscentral.com/how-install-bash-shell-command-line-windows-10">here</a>.
***
## Universal & Essential Tools: 

### Azure CLI - Command Line Interface to manage your Azure datacenter. 

Azure CLI is THE tool to interact with Azure from the command line. You can use to create objects such as virtual machines, networks, scale-sets & clusters, storage, disks, and so on. Azure CLI download and instructions are available <a href="https://docs.microsoft.com/en-us/cli/azure/install-azure-cli">here.</a>

Note that you can also use <a href="https://azure.microsoft.com/en-us/features/cloud-shell/">cloud shell</a> from the <a href="https://portal.azure.com">azure portal</a>.

### Storage Explorer - Utility to help you interact with Azure Storage & Your Data

StorageExplorer is a feature-rich graphical tool for managing Azure Storage resources & data securely. Grab it from <a href="http://storageexplorer.com">storageexplorer.com</a>

* Copy Data to and from Azure, and across Azure storage containers & subscriptions. 
* Share datasets and collaborate with your colleagues inside & outside your organization

### AzCopy - Utility for copying large datasets in and out of Azure efficiently. 
Azcopy is a utility for copying and moving files around to and from Azure or between Azure storage containers. It's more effiicent at copying large datasets comprised or large files or millions of small files, and can easily be incorporated into scripts. 

* <a href="https://docs.microsoft.com/en-us/azure/storage/storage-use-azcopy">Download instructions for AzCopy for Windows.</a>
* <a href="https://docs.microsoft.com/en-us/azure/storage/storage-use-azcopy-linux">Download instructions for AzCopy for Linux.</a>

***
## Azure Batch Tools

These days most of what you need for the common Azure Batch tasks is included in the Azure Batch CLI (see above) under the "az batch" sub-command. However, for the labs we will need some extra tools normally reserved for black belts: 

1) **Batch Labs**
Batch Labs is THE graphical tool for interacting with Azure Batch, and replaces the old Batch Explorer tool. You can <a href="https://azure.github.io/BatchLabs/">download the latest build for your operating system</a>.Batch Labs source code is also online <a href="https://github.com/Azure/BatchLabs">here</a>.

2) **Azure Batch Shipyard**
Batch Shipyard is a set of client-side tools that makes running container based Big Compute jobs on Azure Batch a breeze. Information and overview <a href="https://azure.github.io/batch-shipyard">here</a>. Clone the <a href="https://github.com/Azure/batch-shipyard">github repository</a> to your workstation and follow the install instructions linked on the main README. 

3) **Azure Batch CLI Extensions (Preview)**
The <a href="https://github.com/Azure/azure-batch-cli-extensions">Batch CLI Exensions</a> provide a bunch of new advanced functionality for Azure Batch from the CLI including shipyard style templates, Task Factory, automated file upload/download and new package management features. After installing Azure Batch CLI, follow the instructions on the github README to install the extensions. 

***

## Universal Azure Automation & Development Tools
1) Github & ARM Templates

In case you hadn't heard, infrastructure in the cloud is nothing but CODE. In Azure we call it ARM - Azure Resource Manager. 

Get yourself a <a href="https://github.com/join">github account</a>. You'll find lots and lots of example templates including for HPC. Some examples:


2) Visual Studio Code & Visual Studio
