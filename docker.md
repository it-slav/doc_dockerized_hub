This image can be started in  the following ways

# for interactive mode
docker run -ti -h $(hostname) --name=hub -p 4181:4181 -p 8080:8080 -p 7081:7081 -p 55436:55436 -p 8009:8009 -p 8993:8993 -p 8909:8909 blackduckhub/hub

# detached mode
docker run -d -h $(hostname) --name=hub -p 4181:4181 -p 8080:8080 -p 7081:7081 -p 55436:55436 -p 8009:8009 -p 8993:8993 -p 8909:8909 blackduckhub/hub

See to it that at least the port 8080 is not blocked.

note : 
once started you can connect with you browser on port 8080 it will request for a key enter they key you got from Black Duck software.
this version is not capable of persisting data , once container is stopped the BOM's will be gone

# Instructions for running with docker toolbox

The following instruction will create a docker-machine with the name dockerDemo
1.	First you have to create a docker-machine
2.	Start a mac terminal
Start windows cmd

3.	Of course it depends on the machine you’re on but the following command creates a docker-machine with 12GB ram (which is advised for hub) , 4 cpu’s , and 20GB disk.
4.	docker-machine create -d virtualbox --virtualbox-disk-size "20000" --virtualbox-cpu-count "4" --virtualbox-memory "12288" dockerDemo

5.	once the docker-machine is started get the terminal in the docker-machine context with the following line command
mac : eval $(docker-machine env dockerDemo)
windows : FOR /f "tokens=*" %i IN ('docker-machine env --shell cmd dockerDemo') DO %i

6.	then execute the following command to arrange the virtual box port forwarding , we need to make it accessible on the mac
	VBoxManage controlvm dockerDemo natpf1 "http,tcp,,8080,,8080"
	VBoxManage controlvm dockerDemo natpf1 "glu,tcp,,7081,,7081"
Windows:
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" controlvm dockerDemo natpf1 "http,tcp,,8080,,8080"
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" controlvm dockerDemo natpf1 "glu,tcp,,7081,,7081"

8	docker pull blackduckhub/hub
9.	Now we are ready to start the hub
10.	if you want to see what’s happening it’s best to start it with -ti option like this
        mac :
	docker run -ti -h $(hostname) --name=hub --volumes-from hub -p 4181:4181 -p 8080:8080 -p 7081:7081 -p 55436:55436 -p 8009:8009 -p 8993:8993 -p 8909:8909 blackduckhub/hub
        windows:
         docker run -ti -h %COMPUTERNAME% --name=hub --volumes-from hub -p 4181:4181 -p 8080:8080 -p 7081:7081 -p 55436:55436 -p 8009:8009 -p 8993:8993 -p 8909:8909 blackduckhub/hub
	
11.	After a few minutes you can follow the startup by opening a browser and type the url : http://localhost:7081
12.	Click on the top url which will link you to the glu console.
13.	type : admin, admin
14.	click : deployment at the blue bar.
15.	You will see what is starting.
	
16.	Once started you can connect to the hub with the following url : http://localhost:8080
17.	The login screen should be in your browser.
put in userid : sysadmin , password : blackduck
When you login you will get a screen where you can fill in the key you got from blackduck

# Persisting the BOM (Bill of Material) reports

To persist the BOM's we have to create a data container first.
This is done by executing the following command:
 
 docker run -v /var/lib/blckdck/hub  --name hub_data blackduckhub/hub true

with the "docker ps -a" command you will see there is a stopped container hub_data
Now we start the hub with the --volumes-from command to mount the data container volume in this volume.

docker run -ti  -h $(hostname) --volumes-from hub_data --name=hub --rm -p 4181:4181 -p 8080:8080 -p 7081:7081 -p 55436:55436 -p 8009:8009 -p 8993:8993 -p 8909:8909 blackduckhub/hub

or in detach mode

docker run -d  -h $(hostname) --volumes-from hub_data --name=hub --rm -p 4181:4181 -p 8080:8080 -p 7081:7081 -p 55436:55436 -p 8009:8009 -p 8993:8993 -p 8909:8909 blackduckhub/hub

to scan you can do the following

docker pull blackduckhub/hub

docker run -ti -h $(hostname) -v /var/run/docker.sock:/var/run/docker.sock --rm blackduckhub/scan_container /scanner -h <ip address of your machine>   -p <port nr> -s [http|https] -u <username> -i <image name>

example:

 docker run -ti -h $(hostname) -v /var/run/docker.sock:/var/run/docker.sock --rm blackduckhub/scan_container /scanner -h 192.168.178.15 -p 8080 -s http -u sysadmin -i debian:latest

note:
Keep in mind when you remove the hub_data container you will remove your reports

