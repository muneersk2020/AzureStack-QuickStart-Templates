# Minikube on AzureStack
This template deploys a Ubuntu 16.04 virtual machine on AzureStack running Minikube to manage kubenetes cluster.

## Prerequisites
Follow the below links to create/download an Ubuntu 16.04 LTS Image and upload the same to Azure Stack's Platform Image Repository(PIR)
1. https://azure.microsoft.com/en-us/documentation/articles/azure-stack-linux/
2. https://azure.microsoft.com/en-us/documentation/articles/azure-stack-add-image-pir/
	Note: please use the default values for linuxPublisher,linuxOffer,linuxSku,linuxVersion found in azuredeploy.json while creating the manifest.json in PIR

## Deploying from Portal
``` diff
+ Running into issues? Check out FAQ section for known issues/workarounds.
```
+	Login into Azurestack portal
+	Click "New" -> "Custom" -> "Template deployment -> "Edit Template" -> "Load File" -> Select azure.deploy.json from the local drive -> "Save"
+ Click "Edit Parameters" and 	Fill the parameters. Please note down the admin name and password for later use
+	Select "Create new" to create new Resource Group and give a new resource group name
+	Click "Create"
+ Wait until the template deployment is completed

## Deploying from PowerShell

Download azuredeploy.json and azuredeploy.azurestack.parameters.json to local machine 

Modify parameter value in azuredeploy.parameters.json as needed 

Allow cookies in IE: Open IE at c:\Program Files\Internet Explorer\iexplore.exe -> Internet Options -> Privacy -> Advanced -> Click OK -> Click OK again

Launch a PowerShell console

Change working folder to the folder containing this template

```PowerShell

# Add specific Azure Stack Environment 

Add-AzureRmEnvironment -Name "AzureStackUser" -ArmEndpoint "https://management.local.azurestack.external"
$TenantID = Get-AzsDirectoryTenantId -AADTenantName "YOUR_AAD_TENANT_NAME" -EnvironmentName AzureStackUser
$UserName='YOUR_AZs_TENANT_USER_NAME'
$Password='YOUR_AZs_TENANT_PASSWORD'| ConvertTo-SecureString -Force -AsPlainText
$Credential= New-Object PSCredential($UserName,$Password)
Login-AzureRmAccount -EnvironmentName "AzureStackUser" -TenantId $TenantID -Credential $Credential 
Select-AzureRmSubscription -SubscriptionId "YOUR_SUBSCRIPTION_ID"

$resourceGroupName = "minikuberg"
$resourceGroupDeploymentName = "$($resourceGroupName)Deployment"

# Create a resource group:
New-AzureRmResourceGroup -Name $resourceGroupName -Location "local"

# Deploy template to resource group: Deploy using a local template and parameter file
New-AzureRmResourceGroupDeployment  -Name $resourceGroupDeploymentName -ResourceGroupName $resourceGroupName `
                                    -TemplateFile "LOCAL_PATH_TO azuredeploy.json" `
                                    -TemplateParameterFile "LOCAL_PATH_TO azuredeploy.parameters.json" -Verbose


```

## About Minikube
Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a VM on your laptop for users looking to try out Kubernetes or develop with it day-to-day. It allows to run a very simplistic one node Kubernetes cluster on a Linux VM. It???s the fastest and most straight forward way to get a fully functional Kubernetes cluster running in no time. It allows developers to develop and test their Kubernetes based application deployments on their local machines. 
Architecturally, Minikube VM runs both Master and Agent Node Components locally.
* Master Node components such as API Server, Scheduler, etcd Server, etc are ran in a single uber Linux process called LocalKube. 
* Agent Node components are ran inside docker containers - exactly as they would run on a normal Agent Node. Hence, from a application deployment standpoint, the application does not see any difference when it is deployed on a Minikube or regular Kubernetes cluster.

Here is a brief overview of the minikube deployment on azurestack
![Image of Minikube architecture](https://github.com/vpatelsj/AzureStack-QuickStart-Templates/blob/master/101-vm-linux-minikube/images/minikube.png)



. Our template installs following components:

* Ubuntu 16.04 LTS VM
* Docker-CE from https://download.docker.com/linux/ubuntu 
* Kubectl from https://storage.googleapis.com/kubernetes-release/release/v1.8.0/bin/linux/amd64/kubectl
* Minikube from https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
* xFCE4
* xRDP

## Getting Started
+ This template deploys a linux virtual machine and generates a PublicIP resource so that users can RDP to it. 
  ![Image of Minikube ResourceGroup](https://github.com/vpatelsj/AzureStack-QuickStart-Templates/blob/master/101-vm-linux-minikube/images/ResourceGroup.PNG)

+ To find out the public IP of the virtual machine, open the resource group and click on the virtual machine of the Resource Group generated by the template.
  ![Image of Minikube Public IP](https://github.com/vpatelsj/AzureStack-QuickStart-Templates/blob/master/101-vm-linux-minikube/images/PublicIP.PNG)
  
+ Open Remote Desktop and connect to the virtual machine using the public IP. Enter the user name and password that were entered while creating the resource group.
  
  ![Image of Remote Desktop](https://github.com/vpatelsj/AzureStack-QuickStart-Templates/blob/master/101-vm-linux-minikube/images/RemoteDesktop.PNG) 
  ![Image of Remote Desktop Cred](https://github.com/vpatelsj/AzureStack-QuickStart-Templates/blob/master/101-vm-linux-minikube/images/RemoteDesktopCred.PNG)

+ Open Terminal and enter following command to start minikube
  ![Image of Terminal](https://github.com/vpatelsj/AzureStack-QuickStart-Templates/blob/master/101-vm-linux-minikube/images/terminal.PNG)
```
sudo minikube start --vm-driver=none
sudo minikube addons enable dashboard
sudo minikube dashboard --url
```

+ Open browser and visit the url to see the kubernetes dashboard. Congratulations, you now have a fully working kubernetes installation using minikube.

## Deploying Applications
If you would like to deploy a sample application, please visit the official documentation page of kubernetes, skip the ```"Create Minikube Cluster"``` section as you have already created one above. Simply jump to the section ```"Create your Node.js application"```  at https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/

## FAQ
### RDP Does not work
+ We install xRDP on the Ubuntu VM to enable RDP access. Some users have reported that xRDP takes upto 5minutes to come up and start listening on the port 3389 after deployment is reported completed. During the first 5 minutes after deployment completes, you might experience RDP connection issues. In that case wait for 5 minutes and try again.

+ We have also seen instances where RDP login screen gets stuck at message: "Connecting to port 3350". While we are still trying to understand the root cause of this failure, there is a workaround identified that is helps eradicate this error. Open ssh Terminal to the VM using putty or any of your fav ssh client and issue following commands and retry RDP to the VM again.

```
sudo echo xfce4-session >~/.xsession
sudo service xrdp restart
```

### Dashboard does not come up
+ Dashboard is a service that gets deployed when we issue the command "sudo minikube addons enable dashboard". It takes a bit of a time until the container image gets downloaded and the pod gets fired up when dashboard is enabled for the first time. Until the dashboard service is not up, "sudo minikube dashboard --url" will return error. It shouldn't take more than few seconds the dashboard service to come up.

### Firefox Certificates error
+ Minikube generates all the necessary certificates required to communicate with the Kubernetes cluster. If you run into certificates problem, it could point to faulty minikube startup. Try deleting cluster and start minikube again using following commands:

```
sudo minikube delete
sudo minikube start --vm-driver=none
```
```
