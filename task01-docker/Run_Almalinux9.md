# How to Run an AlmaLinux 9 Container Using Docker  

Make sure that Docker is correctly installed by following the instructions in the [Docker_Windows_Installation.md](https://github.com/filipponicolasi/CompPhysStud-PhD/blob/main/task01-docker/Docker_Windows_Installation.md) file in this repository.  

## 1. Pull the AlmaLinux 9 Image  
Open the Windows terminal and type:  
```cmd
docker pull almalinux:9
```
The system will start downloading the required files. Once the process is complete (after a few seconds), you should see output similar to this:
```
9: Pulling from library/almalinux
b73550e68465: Pull complete
Digest: sha256:787a2698464bf554d02aeeba4e0b022384b21d1419511bfb033a2d440d9f230c
Status: Downloaded newer image for almalinux:9
docker.io/library/almalinux:9
```
To verify the success of this process, open Docker Desktop and check the Images tab. You should see AlmaLinux 9 listed there.

Note: Docker may be running in background, so if you don't see the main window, click on the whale-like icon in the Windows taskbar (under hidden icons), usually located at the bottom right of the screen.

## 2. Start an AlmaLinux 9 Container
To start an AlmaLinux 9 container, type the following command in the Windows terminal:
```cmd
docker run -it --name ContainerName almalinux:9 bash
```
This command:
* Assigns the name *ContainerName* to a new AlmaLinux 9 container
* Launches the container in interactive mode (`-it`)
* Runs the `bash` shell inside the container

Once inside the AlmaLinux 9 environment, your terminal prompt should look something like this:
```charp
[root@23b25a9c8ba1 /]#
```
To verify that you are inside the AlmaLinux 9 container, type:
```cmd
cat /etc/os-release
```
The output should resemble:
```
NAME="AlmaLinux"
VERSION="9.5 (Teal Serval)"
ID="almalinux"
....
....
```

## 3. Exiting and Managing Containers
To exit the container, type:
```cmd
exit
```
You can manage containers by their assigned *ContainerName*.
* Access a running container:
```cmd
docker exec -it ContainerName bash
```
* Stop a running container:
```cmd
docker stop ContainerName
```
* Start a stopped container:
```cmd
docker start ContainerName
```
* List all containers (running and stopped):
```cmd
docker ps -a
```
Alternatively, you can view and manage containers in Docker Desktop under the Containers tab.

Now you are ready to use your AlmaLinux 9 container!
