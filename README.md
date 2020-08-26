
# docker_containers

Docker is cool. Let's do Rust with docker.

## WSL2

On my Win10 I have WSL2 - Windows subsystem for Linux. 
I had WSL 1 for a while, but it was not a true Linux kernel and Docker did not work with WSL 1.
WSL2 is a revolution. With it I have a lightweight virtual machine with a true Linux kernel. Great ! 
Win10 must be version 1903 or higher. My machine did not automatically update to that yet, so I needed the workaround.
I needed to register in the Microsoft Insiders program.  
<https://insider.windows.com/en-us/getting-started>
It uses the same account you use for other Microsoft service.
Now in `Check for updates` I can see the update to version 19041. Just do it!  
Installing WSL2:  
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
I will try to make applications that are separated into server/backend and client/front part. The server will work on Linux. It means everywhere. The server can be on the same machine, in a virtual machine, in the WSL2, on another machine in the local network or anywhere on the internet. So everywhere.
The client/frontend will work inside the browser with Wasm. It means everywhere.  
All will be written in Rust. Full stack Rust.    
It sounds like a good cross-platform solution.  To me.

## Docker

Install Docker with WSL2 support  
<https://docs.docker.com/docker-for-windows/wsl/>
Download and install the stable version from
<https://hub.docker.com/editions/community/docker-ce-desktop-windows/>
Use the whale icon in the notification area to see the containers.
```powershel
# run in powershell to check the installation
docker version
docker run hello-world
```
Docker engine is inside WSL2. Great! 
Windows is only a GUI. Everything really is running in Linux. 
This is the future.

## Example program

We will need git in Linux.
```bash
$ sudo apt install git
$ git --version
$ git config --global user.name "Your Name"
$ git config --global user.email "youremail@yourdomain.com"
```  
Clone this repo:  
`git clone `  


## Container for Rust building

Pull the image for building standalone 100% statically linked app with musl.  
It already has installed everything we need for Rust. Easy.  
<https://blog.sedrik.se/posts/my-docker-setup-for-rust/>
```bash
$ docker pull ekidd/rust-musl-builder
# create an alias to be used easily
$ alias rust-musl-builder='docker run --rm -it -v "$(pwd)":/home/rust/src ekidd/rust-musl-builder'
# Inside the Rust project folder:
# build the project inside the container
$ rust-musl-builder cargo build --release
# let "strip" the binary for minimal size
$ strip target/x86_64-unknown-linux-musl/release/hello
# from 3Mb to 300kb ?!
# create a minimal docker image with the binary file
$ docker build -t scratch_hello .
# run the containerized app
$ docker run --rm scratch_hello
```