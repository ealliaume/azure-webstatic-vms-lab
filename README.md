# Azure - Basics - Lab

This lab is part of the [foundations training](https://github.com/ealliaume/foundations) at [OCTO Technology Australia](http://careers.octo.com.au/).

## Overview of this lab

In this quick lab, we will play with the Azure.

At the end of the lab you will be able to:
- Host a **static website** on Azure
- Create **Load Balanced VMs**

This lab should take approximately 45 minutes.
You will need to use your own azure cloud account.

## The Lab - step by step

### 1. Create an Azure Resource Group

Resource groups in Azure is an approach to group a collection of assets in logical groups.
One benefit of using RGs in Azure is grouping related resources that belong to an application together, as they share a unified lifecycle from creation to usage and finally, de-provisioning.

* Create an Azure Resource Group for this lab (Azure Console Menu > Resources Group > Add)
  * Name: azure-lab
  * Subscription: Pay-As-You-Go
  * Resource Group Location: Australia Southeast


### 2. Host a static website using Azure Blob Storage

Azure Storage could be compared to AWS S3 or GCP Cloud Storage in a way, but doesn't have the same list of features.
For example, a general-purpose v2 storage account provides access to all of the Azure Storage services: blobs, files, queues, tables, and disks.

* Create an Azure Storage Account (Azure Console Menu > Storage accounts > Add)
  * Select the "azure-lab" Resource Group we just created
  * Name: pick something unique like 'ealliaumeazurelab'
  * Location: Australia Southeast
  * Account Kind: Storage V2
  * Replication: Locally-redundant storage
  * Access tier: Hot

* Create a Blob Container for your Storage Account
  * Click on the Storage Account you just created, then on "Blobs" menu item
  * Click on "+ Container" to create a new container and "testblobs" and set the **Public Access Level** to **"Container"**

* Upload a simple HTML page to this container
  * Create a simple "index.html" page on your local system, or upload this one from this repo (TODO)
  * Click on the blob container you just created, then "Upload" and select your index.html file

* Now your file is uploaded, you should be able to see it publically from any browser
  * Click on the file you just uploaded
  * Copy/Paste the URL then put is on a new Incognito window of you browser
  * Notice the pattern of the file https://&lt;YourStorageAccount>.blob.core.windows.net/&lt;YourBlobContainer>/index.html

If you try to remove the "index.html" from the URL, a ResourceNotFound error will be display. So far we gave the public access to the file and container, but the container doesn't know how which file you want to expose.

* Enable **static website option** for your container
  * Go back to the Overview of you Storage Account
  * In the sub menu, click on **Static Website**
  * Enable the feature, put "index.html" in the Index Document Name field, then copy the primary endpoint and open it in your browser... FLOP! It is not working... When using the web site hosting feature, Azure is creating a new "$web" blob container for you, you can't use an existing one, so let's fix that.
  * Go the this new **$web** blob container created, upload you index.html file in it and then now it is working :)


### 2. Create Load Balanced VMs

#### Create instance template for Sydney

Instance templates (Compute engine > Instance Templates):

We will launch our instance from an Instance Template, we will need two of these because there are two bootstrap actions (one to set up the sydney instance, and one for belgium)

* 1 CPU (default)
* Debian (default)
* Firewall > Allow HTTP Traffic
* Startup script (Automation > Startup script): `wget https://storage.googleapis.com/era-boot/frontend-sydney.py
sudo python frontend-sydney.py &`
* Leave all other options as default

#### Create the second instance template for Belgium using “copy”

Click on the template and then use the 'copy' option from the toolbar, this will load the previous selections. Then just change the startup script as below.

`wget https://storage.googleapis.com/era-boot/frontend-belgium.py
 sudo python frontend-belgium.py &`

#### Create instance Group for Sydney

Instance Groups (Compute Engine > Instance Groups)

Create an instance group with the following settings (leave as default those not mentioned)

* Single Zone
* Region: Sydney
* Select the correct instance template
* Autoscaling: ON
* Autoscaling: CPU usage!
* Healthchecks: create a new healthcheck, Protocol:  HTTP 80, Health critieria: 5 5 2 2
* Initial Delay: 120s

#### Create Client Instance Syd

Click on instance template > create VM to create a new VM from the template
The point of creating client instances is to be able to ssh in to these clients and call the backend service from each of these two regions.

* Name: era-client-syd
* Machine type: Micro
* Region: Sydney
* Allow HTTP traffic


#### Create Client Instance Belgium

* Name: era-client-bel
* Machine type: Micro
* Region: Belgium
* Allow HTTP traffic

#### SSH on both Client instance using interface

(VM Instances > Connect > SSH)

* Curl internal IP of instance group
  * curl http://10.152.0.2/
  
You should see 'Hello from Sydney' and 'Bonjour from Europe' from the different instances

### 3. Set up Load Balancers

#### Create HTTP Load Balancer

(Network services > Load Balancer)

* Create a http load balancer and assign a name.

* Create a backend service and assign the details of your instance group.
  * Add healthcheck group
  * Add instance group
  * Set CPU utilisation to 60%

* Don't add anything for Host and Path Rules

* Create Frontend configuration
  * Era-front-1
  * Http
  * Ip v4
  
Review, finalise and create. This will take about 5 minutes!

Check the load balancer has been created (swap with your IP):
`curl http://35.241.43.206/`

#### Check Monitoring Group > Monitoring Tab

### 4. Autoscale!

#### Play with Autoscaling

Do this with the following command (change IP address):

* Then raise the CPU from local shell
* `while true; do curl http://35.241.43.206/service ; done`
* Check monitoring  => menu instances
* Then kill bash loop
* Wait for the cluster to calm down

#### Create instance group for Belgium

* Simple Zone
* Region: Belgium
* Select good instance template
* Autoscaling: ON
* Autoscaling: CPU usage!
* Healthchecks:   era-hc    HTTP 80   /   5 5 2 2
* Initial Delay: 120s

#### Add the new Instance Group to the Load Balancer

`gcloud compute backend-services add-backend era-backend-bel-42 --instance-group=era-ig-bel --region=europe-west1`

* You now have 1 Load balancer service with 2 backends!

## Success!!!

Be sure to delete all resources used!

<img src="./static/success.gif" width="500" />
