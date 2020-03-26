# Create a Minecraft RLCraft server with GCP

In this guide, you will learn to create a RLCraft server for free using Google Cloud Platform (GCP). 
- How it can be for free ? <br/>

Because GCP gives you 300$ free credit for a year, to test their product. You will be able to run you server with it. However, to access the 300$ free credit, GCP asks for a credit card. No money will be charged on this card as long as you have not spent all of your 300$ credit. Google Cloud will paused all running instances once your 300$ credit have been used, and ask if you want to upgrade to a paid account.  

Youtube video :

1. Create a GCP account and access the 300$ free credit
2. Create Kubernetes Engine, one node with 7.5G RAM and 2 vCPU
3. Access the VM, create path to files to your server
4. Download RLCraft and Forge 1.12.2:2838 server files on the VM
5. Access Cloud Run, get deploy.yaml and "kubectl create -f deploy.yaml"
6. Get the public IP address and **Enjoy** 
7. Do a backup and restore a backup
<br/><br/>
8. Going deeper : how did I get this deploy.yaml

## 1. Create a GCP account and access the 300$ free tier

You will require a gmail address. Create one if you don't have.<br/>
Fill the form to access GCP 300$ credit, as an individual.<br/>
It will require a credit card.

<img src="images/free-trial.png"
     alt="Free trial page"
     style="float: left; margin: 10px;" />

Once done, check on GCP that you have gaved access to the 300$ free credit.

<img src="images/free-trial-2.png"
     alt="Free trial page"
     style="float: left; margin: 10px;" />


More info on GCP free trial : https://www.youtube.com/watch?v=P2ADJdk5mYo

## 2. Create Kubernetes Engine, one node with 7.5G RAM and 2 vCPU

## 3. Access the VM, create path to files to your server

`sudo su --`<br/>
`pwd`<br/>
<aa style="margin-left:20px;">/home/your-account-name/</aa><br/>
`cd ..`<br/>
`mkdir rlcraft-server`<br/>
`cd rlcraft-server`<br/>

## 4. Download RLCraft and Forge 1.12.2:2838 server files on the VM

`pwd`<br/>
<aa style="margin-left:20px;">/home/your-account-name/rlcraft-server</aa><br/>
`wget https://media.forgecdn.net/files/2780/296/RLCraft+Server+Pack+1.12.2+-+Beta+v2.5.zip`<br/>
`sudo python`<br/><br/>
`>>> from zipfile import ZipFile`<br/>
`>>> zip_file = ZipFile('/home/your-account-name/rlcraft-server/RLCraft+Server+Pack+1.12.2+-+Beta+v2.5.zip', 'r')`<br/>
`>>> zip_file.extractall('/home/your-account-name/rlcraft-server/')`<br/>
`>>> exit()`<br/><br/>
`mv 'RLCraft Server Pack 1.12.2 - Beta v2.5' data`<br/>
`cd data`<br/>
`wget https://files.minecraftforge.net/maven/net/minecraftforge/forge/1.12.2-14.23.5.2838/forge-1.12.2-14.23.5.2838-installer.jar`<br/>
`wget https://files.minecraftforge.net/maven/net/minecraftforge/forge/1.12.2-14.23.5.2838/forge-1.12.2-14.23.5.2838-universal.jar`<br/><br/>
`cd ..`<br/>
`chmod 777 -R data`<br/>
## 5. Access Cloud Run, get deploy.yaml and "kubectl create -f deploy.yaml"

## 6. Get the public IP address and **Enjoy** 

In Kubernetes engine, connect to your cluster with Cloud Shell<br/>
`kubectl get all`<br/>
`wget https://raw.githubusercontent.com/dleurs/rlcraft/master/deploy.yaml`<br/>
`kubectl create -f deploy.yaml`<br/>
`kubectl get all`<br/>

## 7. Do a backup and restore a backup

### Do a backup :
`NAMESPACE=default`<br/>
`kubectl get pod`<br/>
`POD_ID=lionhope-387ff8d-sdis9`<br/>
`kubectl exec --namespace ${NAMESPACE} ${POD_ID} rcon-cli save-off`<br/>
`kubectl exec --namespace ${NAMESPACE} ${POD_ID} rcon-cli save-all`<br/>
`kubectl cp ${NAMESPACE}/${POD_ID}:/data/world .`<br/>
`kubectl exec --namespace ${NAMESPACE} ${POD_ID} rcon-cli save-on`<br/>

`tar -czvf world-2020.03.26.12h30.tar world/`<br/>

### Do a backup :

You can now connect to your VM, click the gear, import the world.tar
`tar -xzvf world-2020.03.26.12h30.tar`<br/>
`chmod 777 -R world/`<br/>
You can now move this folder into home/rlcraft-server/data/ 

## 8. Going deeper : how did I get this deploy.yaml

- https://github.com/helm/charts/tree/master/stable/minecraft
- https://github.com/itzg/docker-minecraft-server

### Steps to get the deploy.yaml

Install helm 3<br/>
helm repo add stable https://kubernetes-charts.storage.googleapis.com<br/>
helm show values stable/minecraft > config.yaml<br/>
Modify the config file : <br/>
- EULA : true
- put RAM and CPU as needed
- ... <br/>

helm template -f config.yaml mine-release stable/minecraft > deploy.yaml<br/>
Modify the deploy.yaml : <br/> 
- remove Secret
- remove PersistentVolumeClaim
- in the Deployment, change PersistentVolumeClaim to hostPath (as we are running in a single node, no problem to use hostPath) and provide the path

Forge server download (1.12.2 2838) :
- https://files.minecraftforge.net/maven/net/minecraftforge/forge/index_1.12.2.html

RLCraft server download (1.12.2 v2.5) :
- https://www.curseforge.com/minecraft/modpacks/rlcraft/files/2780296