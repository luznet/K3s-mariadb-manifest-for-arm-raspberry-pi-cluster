Eventually I've gotten rid off raspbian in favor of Ubuntu light and currently I'm using dedicated 64bit Mariadb for ARM64

# K3s-mariadb-manifest-for-arm-raspberry-pi-cluster
After almost a day of struggle with mariadb installation on my k3s cluster I decided to share my findings so maybe someone else will benefit from it :):)


# Due to errors while trying to install mysql and/or mariadb using helm I had to create my own manifest and pull images from dockerhub plus add it to local repo inside my k3s cluster .

1.) On your local master(or wherevever you have your docker installed) login to dockerhub using following command:

docker login --username "your-id" (I believe it should be 4-12 digit number)

~ $ sudo docker pull linuxserver/mariadb
i
~ $ sudo docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
linuxserver/mariadb   latest              cce54f9cdf18        2 days ago          278MB

2.) then you need to save your image to a tar file

~ $ sudo docker save linuxserver/mariadb -o $HOME/images/mariadb.tar


3.)now depending on where you have your docker you need to either transfer your tar to a master node or wherever you may use k3s command and run:


~ $ sudo k3s ctr image import /home/pi/images/mariadb.tar

verify

~ $ sudo k3s ctr images list|grep mariadb
docker.io/linuxserver/mariadb:latest        application/vnd.oci.image.manifest.v1+json                sha256:acbba7f612f24e2d9ac36911709ab632077f810bd193b8bb85284f8148989873 271.5 MiB linux/arm/v7                                                                                          io.cri-containerd.image=managed


4.) Now once you have your image in place you may create your storage volumes and create your mariadb using manifest I added into this repository

a.) create a secret with your credentials:

kubectl create secret generic mariadb --from-literal=mariadb-password='mypass' --from-literal=mariadb-root-password='rootpass'

b.) Create a volume

kubectl create -f mariadbpv.yaml

c.) create a volumeclaim

kubectl create -f mariadbpvc.yaml

d.) create mariadb

kubectl create -f mariadb.yaml

5.) Now depending on what you have in your ~/.kube file you either have to run:

kubectl get pods -o wide 

or

kubectl get pods  -n namespace -o wide

In case something doesn't work run 
kubectl describe pods -n namespace podname

you can check what errors it produced


