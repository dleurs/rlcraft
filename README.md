# Create a Minecraft RLCraft server with GCP

In this guide, you will learn to create a RLCraft v2.8.1 server online for free using Google Cloud Platform. 
- How can it be for free ? <br/>

=> Because GCP gives you 300$ free credit for a year, to test their product. You will be able to run your server on it. However, to access the 300$ free credit, GCP asks for a credit card. No money will be charged on this card as long as you have not spent all of your 300$ credit. Google Cloud will paused all running instances once your 300$ credit have been used, and ask if you want to upgrade to a paid account.

Youtube video walkthrough :

1. Create a GCP account and access the 300$ free credit
2. Create Kubernetes Engine, one node with 7.5G RAM and 2 vCPU
3. Access the VM, create path to files to your server
4. Download RLCraft and Forge 1.12.2:2838 server files on the VM
5. Access Cloud Run, get deploy.yaml and "kubectl create -f deploy.yaml"
6. Get the public IP address 
7. Download RLCraft client on Twitch, and Enjoy
8. Do a backup and restore a backup
<br/><br/>
9. Going deeper : how did I get this deploy.yaml

## 1. Create a GCP account and access the 300$ free tier

You will require a gmail address. Create one if you don't have.<br/>
Fill the form to access GCP 300$ credit, as an individual.<br/>
It will require a credit card.<br/>
- https://cloud.google.com/free

<img src="images/free-trial.png"
     alt="Free trial page"
     style="float: left; margin: 10px;" />

Once done, check on GCP that you have gaved access to the 300$ free credit.

<img src="images/free-trial-2.png"
     alt="Free trial page"
     style="float: left; margin: 10px;" />


More info on GCP free trial : https://www.youtube.com/watch?v=P2ADJdk5mYo

## 2. Create Kubernetes Engine, one node with 7.5G RAM and 2 vCPU

Go to GCP, Kubernetes engine<br/>
Create a cluster<br/>
Select Zonal, and check the nearest zone in : <br/>
- https://cloud.google.com/products/calculator

default-pool > 1 node (instead of 3)<br/>
nodes > Select n1, n1-standard-2, **2 vCPU and 7.5 RAM**<br/>
Create<br/>
Wait a few minuts<br/>


## 3. Access the VM, create path to files to your server

On GCP, go to Compute Engine, and connect in SSH, then execute :<br/>
`sudo su --`<br/>
`cd ..`<br/>
`pwd`<br/>
<aa style="margin-left:20px;">/home</aa><br/>
`mkdir rlcraft-server`<br/>
`cd rlcraft-server`<br/>
`mkdir data`<br/>
`cd data`<br/>

## 4. Download RLCraft v2.8.1 and Forge 1.12.2:2838 server files on the VM

`pwd`<br/>
<aa style="margin-left:20px;">/home/rlcraft-server/data</aa><br/>
`wget https://media.forgecdn.net/files/2836/138/RLCraft+Server+Pack+1.12.2+-+Beta+v2.8.1.zip`<br/>
`sudo python`<br/><br/>
`>>> from zipfile import ZipFile`<br/>
`>>> zip_file = ZipFile('/home/rlcraft-server/data/RLCraft+Server+Pack+1.12.2+-+Beta+v2.8.1.zip', 'r')`<br/>
`>>> zip_file.extractall('/home/rlcraft-server/data')`<br/>
`>>> exit()`<br/><br/>
`rm -rf RLCraft+Server+Pack+1.12.2+-+Beta+v2.8.1.zip`<br/>
`wget https://files.minecraftforge.net/maven/net/minecraftforge/forge/1.12.2-14.23.5.2838/forge-1.12.2-14.23.5.2838-installer.jar`<br/>
`wget https://files.minecraftforge.net/maven/net/minecraftforge/forge/1.12.2-14.23.5.2838/forge-1.12.2-14.23.5.2838-universal.jar`<br/><br/>
`cd ..`<br/>
`chmod 777 -R data`<br/>
## 5. Access Cloud Run, get deploy.yaml and "kubectl create -f deploy.yaml"

On GCP, go to Kubernetes Engine, connect to your cluster with Cloud Shell<br/>
`wget https://raw.githubusercontent.com/dleurs/rlcraft/master/deploy.yaml`<br/>
`kubectl create -f deploy.yaml`<br/>
`kubectl get pod`<br/>
`kubectl describe pod release-1-minecraft-5f49ff485d-pv8lq`<br/>
`kubectl logs release-1-minecraft-5f49ff485d-pv8lq --follow`<br/>

## 6. Get the public IP address
`kubectl get service`<br/>

Check how much RAM and CPU the pod is taking:<br/>
`kubectl top pod`<br/>

## 7. Download RLCraft client on Twitch, and Enjoy

### Steps to download RLCraft client, if not already installed :
Download Twitch : <br/>
- https://www.twitch.tv/downloads 

You will require Java 8 (and Oracle force you to create an account to download the jdk)<br/>
- https://www.java.com/fr/download/

Access Twitch, go to mods, Minecraft, Browse modpacks, download RLCraft<br/>
Check that the version is 2.8.1, the same as the server<br/>
<br/>
On the Minecraft Launcher, Edit Profile, in Java Settings : <br/>
Resolution 1600x900<br/>
Make sure *Executable* is pointing to Java 8
For me on macOS, it is : <br/>
`/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/bin/java`<br/> 
JVM Arguments :<br/>
`-Xmx4096m -Xms256m -XX:PermSize=256m -Dfml.ignorePatchDiscrepancies=true -Dfml.ignoreInvalidMinecraftCertificates=true -Duser.language=en -Duser.country=US`<br/>

## 8. Do a backup and restore a backup

### Do a backup :
On GCP, go to Kubernetes Engine, and connect to your cluster using Cloud Shell, then execute :<br/>
`kubectl get pod`<br/>
Copy pod name <br/>
`POD_ID=release-1-minecraft-577889c49d-2xwnp`<br/>
`kubectl exec ${POD_ID} rcon-cli save-off`<br/>
`kubectl exec ${POD_ID} rcon-cli save-all`<br/>
On GCP, go to Compute Engine, and connect in SSH, then execute :<br/>
`sudo su --`<br/>
`cd /home/rlcraft-server/data`<br/>
`now=$(date +'%Y-%m-%d-%Hh%M')`<br/>
`tar -czvf world-${now}.tar world/`<br/>
`cd ..`<br/>
`mkdir backups`<br/>
`mv data/world-${now}.tar backups`<br/>
`cd backups`<br/>
`pwd`<br/>
`ls`<br/>
Click on the gear, download files<br/>

Go back to Kubernetes Engine, and connect to your cluster using Cloud Shell, then execute :<br/>
`kubectl exec ${POD_ID} rcon-cli save-on`<br/>


### Recover a backup :

On GCP, go to Compute Engine, and connect in SSH, then execute :<br/>
`sudo su --`<br/>
`cd /home/rlcraft-server`<br/>
`mkdir backups; cd backups`<br/>
Choose a world, or click on gear and import one from computer
`WORLD=world-2020-03-27-10h02.tar`<br/>
`cd ../data`<br/>
`cp ../backups/${WORLD} .`<br/>
`ls`<br/>

Go to Kubernetes Engine, and connect to your cluster using Cloud Shell, then execute :<br/>
`kubectl get pod --watch`<br/>
`kubectl exec ${POD_ID} rcon-cli stop`<br/>

Go back to Compute Engine :<br/>
`rm -rf world`<br/>
`tar -xzvf ${WORLD}`<br/>
`rm -rf ${WORLD}`<br/>

Not necessary to execute `kubectl exec ${POD_ID} rcon-cli stop`, it will restart automatically

Wait 

## 8. Going deeper : how did I get this deploy.yaml

- https://github.com/helm/charts/tree/master/stable/minecraft
- https://github.com/itzg/docker-minecraft-server

### Steps to get the deploy.yaml

Install helm 3<br/>
`helm repo add stable https://kubernetes-charts.storage.googleapis.com`<br/>
`helm show values stable/minecraft > config.yaml`<br/>
Modify the config file : <br/>
- EULA : true
- put RAM and CPU as needed
- change Liveliness and readiness
- ... <br/>

`helm template -f config.yaml mine-release stable/minecraft > deploy.yaml`<br/>
Modify the deploy.yaml : <br/> 
- remove Secret and PersistentVolumeClaim
- in the Deployment, change PersistentVolumeClaim to hostPath (as we are running in a single node, no problem to use hostPath) and provide the path

Forge server download (1.12.2 2838) :
- https://files.minecraftforge.net/maven/net/minecraftforge/forge/index_1.12.2.html

RLCraft server download (1.12.2 v2.8.1) :
- https://www.curseforge.com/minecraft/modpacks/rlcraft/files/2836138