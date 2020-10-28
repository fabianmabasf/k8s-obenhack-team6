# k8s-obenhack-team6
Repository to share sources for Team 6 of the OpenHack Event

# Challenge 1

docker create network openhack

docker run --name sql6 --network openhack -h sql6 -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=Passwort123' -p 1433:1433 -d mcr.microsoft.com/mssql/server:2017-latest

## New password for User SA


## LogOn to SQL an create DB
docker exec -it sql6 /bin/bash
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P Passwort123
CREATE DATABASE mydrivingDB
SELECT Name from sys.Databases
GO

## Start Dataload
docker run --network openhack -e SQLFQDN=sql6 -e SQLUSER=SA -e SQLPASS="Passwort123" -e SQLDB=mydrivingDB openhack/data-load:v1

#Modify Dockerfile to build Image (not needed)
copy Dockerfile_3
edit Dockerfile
 docker build --no-cache -f Dockerfile -t "tripinsights/poi:1.0" .

docker run --network openhack -d -p 8080:80 --name poi -e "SQL_USER=SA" -e "SQL_DBNAME=mydrivingDB" -e "SQL_PASSWORD=Password123" -e "SQL_SERVER=sql6" -e "ASPNETCORE_ENVIRONMENT=Local" tripinsights/poi:1.0

## Test POI
>curl -i -X GET 'http://localhost:8080/api/poi'

## Login to central Container Registry (ACR)
docker login registrywos0810.azurecr.io -u registrywos0810 -p D955gByCLA8nK+cxw1y4LQrnkwkMZcpF

## Retag image before pushing to ACR
docker tag tripinsights/user-java:1.0 registrywos0810.azurecr.io/tripinsights/user-java:1.0

#Push retaged image to ACR
docker push registrywos0810.azurecr.io/tripinsights/user-java:1.0

docker tag tripinsights/poi:1.0 registrywos0810.azurecr.io/tripinsights/poi:1.0
docker push registrywos0810.azurecr.io/tripinsights/poi:1.0


# Challange 2
 
## Create AKS Cluster with default config
az aks create --resource-group teamResources --name OpenHackAKSCluster --node-count 1 --enable-addons monitoring --generate-ssh-keys
 
## Connect to AKS Cluster
az aks get-credentials --resource-group teamResources --name OpenHackAKSCluster
 
## Create YAML File from existing Images
kubectl run nginx --image=registrywos0810.azurecr.io/tripinsights/userprofile:1.0 --dry-run=client -o yaml > userprofile.yaml
kubectl run nginx --image=registrywos0810.azurecr.io/tripinsights/tripviewer:1.0 --dry-run=client -o yaml > tripviewer.yaml