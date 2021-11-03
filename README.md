## MediaWiki

This project containerizes the Media Wiki application along with Mariadb under RedHat Linux.

## Install

Following are the prerequisite before going forward:
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Helm v3.1.3](https://helm.sh/docs/intro/install/) 
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 

## Docs

### Docker image:
You can use my docker hub images, else you can create you own image beofre create run docker login.

### Use my docker images
- Application image: [anuj23/mediawiki:deploy2](https://hub.docker.com/layers/anuj23/mediawiki/tags/)
- Mariadb Image: [anuj23/mediawiki:db](https://hub.docker.com/layers/anuj23/mediawiki/tags/)

```shell
$ docker pull anuj23/mediawiki:deploy2
$ docker pull anuj23/mediawiki:db
```

### Docker login
If you want to create you own image login to docker hub before proceeding and complete the authentication

```shell
$ docker login
```
### Clone this repo 
```shell
$ git clone https://github.com/anujnarayan23/mediawiki.git
```

### Build application image
```shell
$ cd MediaWiki/Docker/Application
$ docker build -t anuj23/mediawiki:deploy2 .
```

### Build database image 
```shell
$ cd /MediaWiki/Docker/Database
$ docker build -t anuj23/mediawiki:db .
```
## run the GKE cluster command to connect to the cluster

### Create a namespace 
```shell
$ kubectl create ns wm
```

### Install Maridb chart
``` shell
$ cd MediaWiki/Charts/mywiki-db
$ helm upgrade --install mywiki-db -f values.yaml . -n wm
```

### Configure Mariadb one time
Login to Database pod 
``` shell
$ export POD_NAME=$(kubectl get pods --namespace wm -l "app.kubernetes.io/name=mywiki-db,app.kubernetes.io/instance=mywiki-db" -o jsonpath="{.items[0].metadata.name}")
$ kubectl exec -it $POD_NAME -n wm bash
```

Open mysql terminal
```shell
$ mysql
```

Run to give perms to root user
```shell
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'; FLUSH PRIVILEGES;
```
Exit from the pod.

### Mariadb hostname
As we are using type as a Load balancer host will be ip address of the app


### Install the application
```shell
$ cd MediaWiki/Charts/mywiki
$ helm upgrade --install mywiki -f values.yaml . -n wm
```

### Port forward to access from GKE cluter pod
 ```shell
 $ export POD_NAME=$(kubectl get pods --namespace wm -l "app.kubernetes.io/name=mywiki,app.kubernetes.io/instance=mywiki" -o jsonpath="{.items[0].metadata.name}")
 $ kubectl --namespace wm port-forward $POD_NAME 8080:80
```

## Setup the Application 
Open the application on [http://34.82.215.209:8080](http://34.82.215.209:8080)
<table>
  <tr>
    <td><img src="Screenshots/1.JPG"></td>
 </tr>
 <tr>
    <td><img src="Screenshots/2.JPG"></td>
 </tr>
 </table>

### Database hostname
update the hostname to 35.247.72.25

<table>
  <tr>
    <td><img src="Screenshots/3.JPG"></td>
 </tr>
 <tr>
    <td><img src="Screenshots/4.JPG"></td>
 </tr>
 </table>

## Update Application helm chart.
### Download LocalSettings.php
### Update Values.yaml
Update config property in File: [Values.yaml](https://github.com/ksingh3/MediaWiki/blob/3dc5d3103759b4c2bd52975859ec31468606ec89/Charts/mywiki/values.yaml#L40-L138) using the downloaded LocalSettings.php.

## Copy the LocalSettings.php file inside the pod
kubectl cp /root/mediawiki/Charts/mywiki/LocalSettings.php wm/mywikiapp-7c46bfff98-dxg7w:/var/www/html

This time it will mount LocalSettings.php file under `/var/www/html/` using config map.

<table>
  <tr>
    <td><img src="Screenshots/5.JPG"></td>
 </tr>
   <tr>
    <td><img src="Screenshots/6.JPG"></td>
 </tr>
 </table>

