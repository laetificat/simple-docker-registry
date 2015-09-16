# Set up a private Docker registry
This repository is aimed to create an unsecure Docker registry on Fedora, this does not make use of any certificates or security.  
The document is split up into two parts, the first one is the Docker registry, and the second one is the client.  
Things you need to keep in mind:

- Everything is unsecured, no login, no SSL/TLS
- It's aimed to work with Fedora as host OS
- You need to make a small change to every Docker client that wants to connect

## Docker registry
I take you already have a system with Fedora installed and completely updated. So the first command you run is `yum install docker-registry`.
Once it's installed you run `service docker-registry start`, and `systemctl enable docker-registry` if you want the registry to run at startup.  
Now you can check if the registry is running, run the command `curl localhost:5000` and you should get a JSON response from the registry. Next
up we want to open the port so other devices can connect to the registry.

Run the command `firewall-cmd --permanent --zone=public --add-port=5000/tcp` to open up port 5000. (You might want to port forward on your router
to access the registry from the internet, but this is generally not a good idea as anyone can push and pull on this registry.)
Run `systemctl restart firewalld.service` to restart the firewallD service and apply the new rule.

Now you should have the repository all done, but keep in mind it's really unsafe and public.

## Docker client
This part is split up in two sections, the native Docker client, and Boot2docker (or Docker Toolbox). It depends on what system you are running on.

### Native Docker client
Use your favourite package manager, for my instance it's yum. Install Docker according to the official documentation [HERE](https://docs.docker.com/installation/).
Once you got through this part we can go on and configure our client to accept our insecure registry.

Officially you should edit the Docker config where a line exists that will allow you to add extra options, like accepting insecure registry. BUT, for me it did not
work for some reason. So what I did was edit the service file that exists on Fedora under `/usr/lib/systemd/system/docker.service`, in this file I edited the line
where it starts the Docker daemon and added the flag `--insecure-registry [LOCAL IP OF THE REGISTRY]:5000`. I restarted the Docker daemon with `service docker restart`
and it automagically worked!

### Docker Toolbox
At this stage you should have Docker Toolbox installed, if not, you can get it [HERE](https://www.docker.com/toolbox). Once installed we can continue.
By default Docker Toolbox will install a Docker shell environment (client), Kitematic UI, VirtualBox, and Docker Machine. Also Docker Compose if you are running on Mac.
The way Docker Toolbox works for Windows and Mac is that a very super light virtual machine is created in VirtualBox with a Linux OS which contains a native Docker client.
But the way it's built makes it almost transparent in your terminal, you can still browse your local files as if you were outside of the virtual environment.

Now the configuration part, first we want to get rid of the default Docker virtual machine. You can do this by using Docker Machine on your host by running
`docker-machine rm default`. Now we want a new one wich accepts an insecure Docker registry, do this by running the command 
`docker-machine docker-machine create --driver virtualbox --engine-insecure-registry [LOCAL IP OF THE REGISTRY]:5000 default`. And bam, you have a new default machine that
allows your private insecure registry.

You can now push and pull to and from your private registry, for example: `docker tag ubuntu [LOCAL IP OF THE REGISTRY]:5000/ubuntu`. This retagged the official Ubuntu
base image I had to my private registry! Now just run `docker push [LOCAL IP OF THE REGISTRY]:5000/ubuntu` to push it to your registry.
To pull an image you can just use either run or pull (I recommend pull because it will force Docker to fetch the latest changes).
