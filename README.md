# Create a Minecraft RLCraft server with GCP

In this guide, you will learn to create a RLCraft server online for free using Google Cloud Platform (GCP). 
- How it can be for free ? <br/>

=> Because GCP gives you 300$ free credit for a year, to test their product. You will be able to run you server with it. However, to access the 300$ free credit, GCP asks for a credit card. No money will be charged on this card as long as you have not spent all of your 300$ credit. Google Cloud will paused all running instances once your 300$ credit have been used, and ask if you want to upgrade to a paid account.  

Youtube video :

1. Create a GCP account and access the 300$ free credit
2. Create Kubernetes Engine, one node with 7.5G RAM and 2 vCPU
3. Access the VM, create path to files to your server
4. Download RLCraft and Forge 1.12.2:2838 server files on the VM
5. Access Cloud Run, get deploy.yaml and "kubectl create -f deploy.yaml"
6. Get the public IP address 
7. **Enjoy** 
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
nodes > Select n1, n1-standard-2<br/>
security > uncheck Activate integrity surveillancy<br/>
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

## 4. Download RLCraft and Forge 1.12.2:2838 server files on the VM

`pwd`<br/>
<aa style="margin-left:20px;">/home/rlcraft-server</aa><br/>
`wget https://media.forgecdn.net/files/2780/296/RLCraft+Server+Pack+1.12.2+-+Beta+v2.5.zip`<br/>
`sudo python`<br/><br/>
`>>> from zipfile import ZipFile`<br/>
`>>> zip_file = ZipFile('/home/rlcraft-server/RLCraft+Server+Pack+1.12.2+-+Beta+v2.5.zip', 'r')`<br/>
`>>> zip_file.extractall('/home/rlcraft-server/')`<br/>
`>>> exit()`<br/><br/>
`mv 'RLCraft Server Pack 1.12.2 - Beta v2.5' data`<br/>
`cd data`<br/>
`wget https://files.minecraftforge.net/maven/net/minecraftforge/forge/1.12.2-14.23.5.2838/forge-1.12.2-14.23.5.2838-installer.jar`<br/>
`wget https://files.minecraftforge.net/maven/net/minecraftforge/forge/1.12.2-14.23.5.2838/forge-1.12.2-14.23.5.2838-universal.jar`<br/><br/>
`cd ..`<br/>
`chmod 777 -R data`<br/>
## 5. Access Cloud Run, get deploy.yaml and "kubectl create -f deploy.yaml"

## 6. Get the public IP address

On GCP, go to Kubernetes Engine, connect to your cluster with Cloud Shell<br/>
`kubectl get all`<br/>
`wget https://raw.githubusercontent.com/dleurs/rlcraft/master/deploy.yaml`<br/>
`kubectl create -f deploy.yaml`<br/>
`kubectl get all`<br/>
`kubectl get pod`<br/>
`kubectl describe pod release-1-minecraft-5f49ff485d-pv8lq`<br/>
`kubectl logs release-1-minecraft-5f49ff485d-pv8lq --follow`<br/>

## 7. Enjoy

### Steps to download RLCraft client, if not already installed :
Download Twitch : <br/>
- https://www.twitch.tv/downloads 

You will require Java 8 (and Oracle force you to create an account to download the jdk)<br/>
- https://www.java.com/fr/download/

Access Twitch, go to mods, Minecraft, Browse modpacks, download RLCraft<br/>
<br/>
On the Minecraft Launcher, Edit Profile, in Java Settings : <br/>
Make sure *Executable* is pointing to Java 8
For me on macOS, it is : <br/>
`/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/bin/java`<br/> 
JVM Arguments :<br/>
`-Xmx4096m -Xms256m -XX:PermSize=256m -Dfml.ignorePatchDiscrepancies=true -Dfml.ignoreInvalidMinecraftCertificates=true -Duser.language=en -Duser.country=US`<br/>

## 8. Do a backup and restore a backup

### Do a backup :
On GCP, go to Kubernetes Engine, and connect to your cluster using Cloud Shell, then execute :<br/>
`NAMESPACE=default`<br/>
`kubectl get pod`<br/>
`POD_ID=lionhope-387ff8d-sdis9`<br/>
`kubectl exec --namespace ${NAMESPACE} ${POD_ID} rcon-cli save-off`<br/>
`kubectl exec --namespace ${NAMESPACE} ${POD_ID} rcon-cli save-all`<br/>
`kubectl cp ${NAMESPACE}/${POD_ID}:/data/world .`<br/>
`kubectl exec --namespace ${NAMESPACE} ${POD_ID} rcon-cli save-on`<br/>

`tar -czvf world-2020.03.26.12h30.tar world/`<br/>

### Recover a backup :

On GCP, go to Compute Engine, and connect in SSH, then execute :
`tar -xzvf world-2020.03.26.12h30.tar`<br/>
`chmod 777 -R world/`<br/>
You can now move this folder into home/rlcraft-server/data/ 

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
- ... <br/>

`helm template -f config.yaml mine-release stable/minecraft > deploy.yaml`<br/>
Modify the deploy.yaml : <br/> 
- remove PersistentVolumeClaim
- in the Deployment, change PersistentVolumeClaim to hostPath (as we are running in a single node, no problem to use hostPath) and provide the path

Forge server download (1.12.2 2838) :
- https://files.minecraftforge.net/maven/net/minecraftforge/forge/index_1.12.2.html

RLCraft server download (1.12.2 v2.5) :
- https://www.curseforge.com/minecraft/modpacks/rlcraft/files/2780296