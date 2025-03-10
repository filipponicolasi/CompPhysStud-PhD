# How to Run an AlmaLinux 9 Container Using Docker  

Make sure that Docker is correctly installed by following the instructions in the [Docker_Windows_Installation.md](https://github.com/filipponicolasi/CompPhysStud-PhD/blob/main/task01-docker/Docker_Windows_Installation.md) file in this repository.  

Now, open the Windows terminal and type:
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

Note: Docker may be running in the background in Resource Server mode, so if you don't see the main window, click on the whale-like icon in the Windows taskbar (under hidden icons), usually located at the bottom right of the screen.

Now start an Almalinux9 container typing on the windows terminal:
```cmd
docker run -it --name ContainerName almalinux:9 bash
```
this command assigns the name *ContainerName* to a new  almalinux9 container and runs it. Now, we have an almalinux9 environment inside the Terminal. You should see in the terminal a message like `[root@23b25a9c8ba1 /]#`. 

To exit the container insert the command
```cmd
exit
```
To start the container use the previous assigned name inserting the command
```cmd
start -i ContainerName
```
