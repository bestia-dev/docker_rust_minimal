
# docker_rust_minimal

Docker is cool. Let's do Rust with Docker.

## WSL2

On my Win10 I have WSL2 - Windows subsystem for Linux.  
I had WSL 1 for a while, but it was not a true Linux kernel and Docker did not work with WSL 1.  
WSL2 is a revolution. With it I have a lightweight virtual machine with a true Linux kernel. Great !  
Win10 must be version 1903 or higher. My machine did not automatically update to that yet, so I needed the workaround.
I needed to register in the Microsoft Insiders program.  
<https://insider.windows.com/en-us/getting-started>  
It uses the same account you use for other Microsoft service.  
Now in `Check for updates` I can see the update to version 19041. Just do it!  

### Installing WSL2:  

<https://docs.microsoft.com/en-us/windows/wsl/install-win10#update-to-wsl-2>
```powershell
# Open PowerShell as Administrator
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
# restart and update to WSL2
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
# restart your machine. Set WSL2 as default
wsl --set-default-version 2
# I get the error: WSL 2 requires an update to its kernel component. 
# visit https://aka.ms/wsl2kernel and do the update
# I choose to install the Debian distro:
https://www.microsoft.com/sl-si/p/debian/9msvkqc78pk6?rtc=1
# give it a password you will remember
# then in linux bash
$ sudo apt update
$ sudo apt dist-upgrade
```

## Rust

I will try to compile all my Rust code to Linux or Wasm.  
The Windows OS is less and less interesting to me. It is really good for GUI applications. But I am not interested in those, because it limits the use of the program to only that OS. This kind of platform vendor-lock-in because of the GUI is stupid.  
I will try to make applications that are separated into server/backend and client/frontend. The server will work on Linux. It means everywhere. The server can be on the same machine, in a virtual machine, in the WSL2, on another machine in the local network or anywhere on the internet. So everywhere.  
The client/frontend will work inside the browser with Wasm. It means everywhere.  
All will be written in Rust. Full stack Rust.    
It sounds like a good cross-platform solution.  To me.

## Docker

Install Docker in windows with WSL2 support  
<https://docs.docker.com/docker-for-windows/wsl/>  
Download and install the stable version from  
<https://hub.docker.com/editions/community/docker-ce-desktop-windows/>  
Use the whale icon in the notification area to see the Docker containers.
```powershel
# run in powershell to check the installation
docker version
docker run hello-world
```
Docker engine is inside WSL2. Great!  
Windows is only a GUI. Everything really is running in Linux.  
This is the future.

## Container for Rust building

For now we don't need to install Rust on our Debian.  
Just pull the image for Rust building standalone 100% statically linked app with musl.  
It already has installed everything we need for Rust. Easy.  
<https://blog.sedrik.se/posts/my-docker-setup-for-rust/>
```bash
# bash commands
$ docker pull ekidd/rust-musl-builder
# create an alias to be used easily to run the container
$ alias rust-musl-builder='docker run --rm -it -v "$(pwd)":/home/rust/src ekidd/rust-musl-builder'
# docker container can be run for only one execution and then removed
# --rm means the container is removed after execution
# -it means interactive (not in background)
# -v Volume links a read/write host directory to the container file system
# $(pwd) means "Present Working Directory" and has nothing to do with passwords
# docker commands finish with the image name
```

## Git

We will need git in Debian for our example program.
```bash
$ sudo apt install git
$ git --version
$ git config --global user.name "Your Name"
$ git config --global user.email "youremail@yourdomain.com"
```  
Clone this repo:  
```bash
cd ~
mkdir rustprojects
cd rustprojects
git clone git@github.com:LucianoBestia/docker_rust_minimal.git

```  

## Building a CLI program

Our pwd Present Working Directory is the cloned project.
```bash
cd ~/rustprojects/docker_rust_minimal
```
We will use the alias to run a docker container. And after that we will give the command to execute in the container. We want to build the Rust project.

```bash
$ rust-musl-builder cargo build --release
```
That's it. The docker container was spinned, the volume was linked, the project was built. We can find the result in `target/x86_64-unknown-linux-musl/release/hello`.  

## Make it small

Now we have a binary (executable) and we want to deploy it somewhere. 
We will try to make it as small as possible. Maybe that is not smart for every scenario out there, but let's have a look.  

```bash
# let "strip" the binary for minimal size
$ strip target/x86_64-unknown-linux-musl/release/hello
# from 3MB to 300kB ?!
```

## Build the Docker image

We want to use docker to encapsulate all the needed files and configuration so the person on the other side does not need to worry about it.   
We need to have a Dockerfile in our Working Directory. It is already in this git repo. It looks simple like this:
```bash
FROM scratch
COPY target/x86_64-unknown-linux-musl/release/hello /
CMD ["/hello"]
```
`scratch` is the smallest possible docker image base. We can use it because our binary is statically linked to everything including `musl`. The next smallest image base is Alpine Linux. Only 5 MB. The next can be Debian Slim 88 MB. You see the difference in size.  
We need to copy only our binary file to the image.  
When the container is run, it will execute our binary.  
Now build this image:
```bash
$ alias utc='date -u +"%Y-%m-%dT%H-%M-%S"'
$ docker build --no-cache -t scratch_hello -t scratch_hello:$(utc) .
```
For deployment reasons we want to give a version to the image. It is difficult to make this smart. I will just use the UTC date and time.  
`--no-cache` for this example we avoid caching  
`-t` means tag or name.There can be more than one tag curiously.  
`.` means the Dockerfile is here in the Working Directory
We can see it now in our list of local docker images:
```bash
$ docker images
```
It is only 305kB. Woohoo!

## Run the containerized CLI

We can now run the docker container with our CLI just as we would run the app itself:  
```bash
$ docker run --rm scratch_hello
```
`--rm` means remove the container after execution  
We can run it also from the `Windows Command Prompt` and `PowerShell terminal`.
Incredible !

## Deploy

Docker Hub is a website to share docker images easily.  Alternatively it can be saved to a tar file and then transfered, copied and finally loaded.
```bash
$ docker save --output scratch_hello-2020-08-26T11-45-51.tar scratch_hello:2020-08-26T11-45-51
```
On the other side:  
```bash
$ docker load --input scratch_hello-2020-08-26T11-45-51.tar
$ docker tag scratch_hello:2020-08-26T11-45-51 scratch_hello:latest
$ docker run scratch_hello
```
Adding the tag `latest` make it easier to call that image later.  