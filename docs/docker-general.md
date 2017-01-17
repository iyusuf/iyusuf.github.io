# Docker in Windows with Docker Toolbox

After putting off for a long time I have finally began to start playing with Docker. I'm a big fan of virtualization and specially using virtualization technologies to build development sandbox. I predominantly use 

## **Gotchas** 
- When testing using browser please replace "localhost" with the proper docker ip address i.e. "192.168.99.100"
	- If in doubt find docker machine's ip by invoking the following command.
	- ```$ docker-machine ip```
- Careful about file permission. Set it specifically with Dockerfile
- CR LF line feed in Windows environment. Turn these executable and other files into Linux/Unix file format with just LF
- docker-compose.yml file can not have tab.
- docker-compose up may not be able to connect to docker-machine. Execute the following to reinject docker-machine env.
	- eval $("C:\Program Files\Docker Toolbox\docker-machine.exe" env)

# Chapter 5
## Copying "cmd.sh" while building
### Page 80

Be careful about all the relative and absolute path used in Dockerfile and from command line etc. They all mean different things in terms of when they are used. Sometime they mean path on the local host machine, some time they mean path residing at boot2docker VM that act as Docker host machine (but a virtual machine itself) that sits on top of Windows. Finally some time these path will mean paths residing at "container" vms.

Original code from book had "/" root as the destination folder for "cmd.sh" file. Trying to save cmd.sh at root "/" level didn't work. The build worked when "cmd.sh" file was copied to /app folder which is also a work directory.

**Content of Dockerfile **
```
FROM python:3.4

RUN groupadd -r uwsgi && useradd -r -g uwsgi uwsgi
RUN pip install Flask==0.10.1  uWSGI==2.0.8
WORKDIR /app
COPY app /app
COPY cmd.sh /app

EXPOSE 9090 9191
USER uwsgi
 
CMD ["/app/cmd.sh"]
```


**Content of cmd.sh**
```
#!/bin/bash
set -e

if [ "$ENV" = 'DEV' ]; then
	echo "Running Development Server"
	exec python "identidock.py"
else
	echo "Running Production Server"
	exec uwsgi --http 0.0.0.0:9090 --wsgi-file /app/identidock.py --callable app --stats 0.0.0.0:9191
fi
```


Let us ssh into boot2docker the "default" Linux host VM machine in VirtualBox. Let us open Docker console and issue the following command to get into Linux VM that is acting as a Docker container host machine.
```
docker-machine ssh
```

Let us build the image
```
docker build -t identidock .
```

Let us now run a container from **identilock** image in DEV environment
```
docker run -e "ENV=DEV" -p 5000:5000 identidock
```

Note: Both docker "build" and "run" command works successfully using Windows Docker console and from within boot2docker vm. 
Test using a browser.
```
http://192.168.99.100:5000/
```

Let us now run a container from **identilock** image in Production environment by not setting ENV variable and setting ports to 9090 and 9191
```
docker run -e  -p 9090:9090 -p 9191:9191 identidock
```

Note: Both docker "build" and "run" command works successfully using Windows Docker console and from within boot2docker vm. 

Test using a browser. You can run the browser from Windows machine. Note the host address is 192.168.99.100 and not "localhost". Localhost will not work as Docker's simulated Linux host is a VirtualBox virtual machine and it has a specific IP that is assigned to it by Docker(?). We can also find out current IP address for "default". Use command "```docker-machine ip```" from Docker console to find out IP address of "default" vm.

```
http://192.168.99.100:9090/
```

----------------------------------------------------------


# Chapter 5
## Work directory path for Windows host.
### Page 51

Defining mapped workdir from within docker container to Windows folder can be tricky. Making a Windws folder available as work directory requires a few steps. Docker uses Oracle VirtualBox to simulate Linux host. The Docker Linux host is a virtual machine guest on VirtualBox. First a shared folder needs to be defined inside VirtualBox that will point to a Windows local folder. Later the defined folder needs to be mounted from within Docker virtual machine, which shows up as "default" in VirtualBox. Finally the mounted directory inside "default" virtual machine (which is considered host linux machine as far as containers are concerned) will be mapped inside Dockerfile when we will start container using Docker's "run" command.

Let us walk through the full process.

- VirtualBox: 
	- Create Shared folder in VirtualBox
		- Settings --> Shared folder
		- Example
			- "projects" on D:/projects

- Docker VM
	- Start docker console
		- log in to docker vm
		```docker-machine ssh```
		- Make directory projects under /home/docker folder.  
		```docker@default:~$ mkdir projects```

- Mount VirtualBox's "projects" under "Docker VM's /home/docker/projects"
		```docker@default:~$ sudo mount -t vboxsf -o uid=1000,gid=50 projects /home/docker/projects```
	
- Mount point will be lost once docker-machine boot2docker starts, usually named "default" in VirtualBox.
	- [Read more here about how to persist a mount ](https://github.com/boot2docker/boot2docker/blob/master/doc/FAQ.md#local-customisation-with-persistent-partition)
		- Basically you log in to docker-machine vm.
		- Change directory to /var/lib/boot2docker
		- Create a file named ```/var/lib/boot2docker/bootlocal.sh ```
		- Add the following two lines.
		- ``` mkdir -p /home/docker/projects```
		- ``` mount -t vboxsf -o uid=1000,gid=50 projects /home/docker/projects```
		-  Now mount point 'projects' will survive a docker-machine restart.


- Now let us test work volume by running a container from "identilock" app from "Using Docker" book.
	- First let us ssh into boot2dockerCreate Container with shared volume
	- For "using-docker" book's Python app
		- ``` docker run -d -e "ENV=DEV" -p 5000:5000 -v /home/docker/projects/using-docker/identidock/app:/app identidock```
		- ``` docker run -d -p 9090:9090 -p 9191:9191 -v /home/docker/projects/using-docker/identidock/app:/app identidock```
		- Test using a browser ```http://192.168.99.100:500 ```
		- ```$ docker run -d -p 9090:9090 -p 9191:9191 -v /home/docker/projects/using-docker/identidock/app:/app2 identidock```
		
	- **Doesn't work**
		- Can not create a volume with /app since /app is already defined as a working directory? Need to verify.
		- Name besides "app" will work for -v options as container destination.
		- For example the following command set will work. Container's mapped volume is given a name of "app2" which is different from Container's work directory name "app".
		- ```docker run -d -p 9090:9090 -p 9191:9191 -v /home/docker/projects/using-docker/identidock/app:/app2 identidock```

docker build -t webapp --rm=true --force-rm=true .