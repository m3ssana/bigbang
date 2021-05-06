# Big Bang Quick Start


## Overview 
The Big Bang Quick Start is meant to offer an easy to deploy preview of BigBang.       
Note: The current implementation of the Quick Start limits customizability of the BigBang Deployment.      
(It's doing a GitOps defined deployment from a repo you don't control.)


## Step 1. Provision a VM
64GB ram, 16 CPU core, Ubuntu Server 20.04 LTS recommended for demo purposes       
(Ubuntu comes up slightly faster than RHEL, although both work fine)       
You want to have network connectivity to said VM, provisioning with a public IP and a security group locked down to your IP should work. Otherwise a Bare Metal server or even a vagrant box vm configured for remote ssh works fine.       
Note: The quickstart repo's init-k3d.sh, starts up k3d using flags to disable the default ingress controller and map the VM's port 443 to a dockerized LB's port 443, which will eventually map to the istio ingress gateway. That along with some other things (Like leveraging a Lets Encrypt Free Wildcard HTTPS Certificate) are done to lower the prerequisites barrier to make basic demos easier. 


## Step 2. SSH into machine and install prerequisite software
1. Setup SSH
```
[User@Laptop:~]
touch ~/.ssh/config
chmod 600 ~/.ssh/config
cat ~/.ssh/config
temp="""##########################
Host k3d
  Hostname 1.2.3.4  #IP Address of k3d node
  IdentityFile ~/.ssh/bb-onboarding-attendees.ssh.privatekey   #ssh key authorized to access k3d node
  User ubuntu
  StrictHostKeyChecking no   #Useful for vagrant where you'd reuse IP from repeated tear downs
#########################"""
echo "$temp" | sudo tee -a ~/.ssh/config  #tee -a, appends to preexisting config file
```

2. Install docker
```bash
[admin@Laptop:~]
ssh k3d
[ubuntu@k3d:~]
curl -fsSL https://get.docker.com | bash

docker run hello-world
# docker: Got permission denied while trying to connect to the Docker daemon socket at 
# unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.35/containers/create: 
# dial unix /var/run/docker.sock: connect: permission denied.See 'docker run --help'.

sudo docker run hello-world
# If docker only works when you use sudo, further configuration is needed.
# Basically you need to add your non-root user to the docker group.

sudo groupadd docker
sudo usermod --append --groups docker $USER 

# When users are added to a group in linux, a new process needs to spawn in order for the new 
# permissions to be recognised, due to a Linux security feature preventing running processes 
# from gaining additional privileges on the fly. (log out and back in is the sure fire method)
exit
[admin@Laptop:~]
ssh k3d
[ubuntu@k3d:~]
docker run hello-world # validate install was successfull
```

3. Install k3d
```bash
[ubuntu@k3d:~]
wget -q -P  /tmp https://github.com/rancher/k3d/releases/download/v3.0.1/k3d-linux-amd64
mv /tmp/k3d-linux-amd64 /tmp/k3d
sudo chmod +x /tmp/k3d
sudo mv -v /tmp/k3d /usr/local/bin/
k3d --version # validate install was successfull
```

4. Install Kubectl
```bash
[ubuntu@k3d:~]
wget -q -P /tmp "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo chmod +x /tmp/kubectl
sudo mv /tmp/kubectl /usr/local/bin/kubectl
sudo ln -s /usr/local/bin/kubectl /usr/local/bin/k #alternative to alias k=kubectl in ~/.bashrc
k version # validate install was successfull
```

5. Install Terraform
```bash
[ubuntu@k3d:~]
wget https://releases.hashicorp.com/terraform/0.14.9/terraform_0.14.9_linux_amd64.zip
sudo apt update && sudo apt install unzip
unzip terraform*
sudo mv terraform /usr/local/bin/
terraform version # validate install was successfull
```

6. Run OS Preconfiguration 
```bash
[ubuntu@k3d:~]
sudo sysctl -w vm.max_map_count=262144 # for ECK
sudo swapoff -a # Turn off all swap devices and files (won't last reboot)
# For swap to stay off you can remove any references found via 
# cat /proc/swaps
# cat /etc/fstab

# For Sonarqube
sudo sysctl -w vm.max_map_count=524288
sudo sysctl -w fs.file-max=131072
ulimit -n 131072
ulimit -u 8192
```


## Step 3. Clone the Big Bang Quick Start Repo
https://repo1.dso.mil/platform-one/quick-start/big-bang#big-bang-quick-start

1. Clone the repo
```
[ubuntu@k3d:~]
cd ~
git clone https://repo1.dso.mil/platform-one/quick-start/big-bang.git
cd ~/big-bang
```

2. Look up your IronBank image pull credentials from https://registry1.dso.mil
```text
How to Look up IB Image Pull Credentials:
1. In a web browser go to https://registry1.dso.mil
2. Login via OIDC provider
3. Top right of the page, click your name --> User Profile
4. Your image pull username is labeled "Username"
5. Your image pull password is labeled "CLI secret"
(Note: The image pull creds are tied to the life cycle of an OIDC token
which expires after 30 days, so if 30 days have passed since your last 
login to IronBank, the credentials will stop working until you relogin
to the https://registry1.dso.mil GUI)
```

3. Verify your credentials work 
```bash
[ubuntu@k3d:~/big-bang]
docker login https://registry1.dso.mil
# It'll prompt for "Username: " (type it out)
# It'll prompt for "Password: " (copy paste it, or blind type it as it will be masked)
# Login Succeeded
```

4. Create a terraform.tfvars file with you registry1 credentials in your copy of the cloned repo
```bash
[ubuntu@k3d:~/big-bang]
vi ~/big-bang/terraform.tfvars
```

* Add the following contents to the newly created file
```text
registry1_username="REPLACE_ME"
registry1_password="REPLACE_ME"
```


## Step 4. Follow the deployment directions on the Big Bang Quick Start Repo
[Link to Big Bang Quick Start Repo](https://repo1.dso.mil/platform-one/quick-start/big-bang#big-bang-quick-start)


## Step 5. Edit your Laptop's HostFile to access the web pages hosted on the BigBang Cluster
```bash
[ubuntu@k3d:~/big-bang]
k get vs -A  # Short version of, kubectl get virtualservices --all-namespaces
# NAMESPACE    NAME                                      GATEWAYS                HOSTS                          AGE
# monitoring   monitoring-monitoring-kube-alertmanager   ["istio-system/main"]   ["alertmanager.bigbang.dev"]   8d
# monitoring   monitoring-monitoring-kube-grafana        ["istio-system/main"]   ["grafana.bigbang.dev"]        8d
# monitoring   monitoring-monitoring-kube-prometheus     ["istio-system/main"]   ["prometheus.bigbang.dev"]     8d
# argocd       argocd-argocd-server                      ["istio-system/main"]   ["argocd.bigbang.dev"]         8d
# kiali        kiali                                     ["istio-system/main"]   ["kiali.bigbang.dev"]          8d
# jaeger       jaeger                                    ["istio-system/main"]   ["tracing.bigbang.dev"]        8d
```

* Linux/Mac Users:
```bash
[admin@Laptop:~]
sudo vi /etc/hosts
```

* Windows Users: 
1. Right click Notepad -> Run as Administrator
2. Open C:\Windows\System32\drivers\etc\hosts

* Add the following entries to the hostfile, where 1.2.3.4 = k3d VM's IP
```text
1.2.3.4  alertmanager.bigbang.dev
1.2.3.4  grafana.bigbang.dev
1.2.3.4  prometheus.bigbang.dev
1.2.3.4  argocd.bigbang.dev
1.2.3.4  kiali.bigbang.dev
1.2.3.4  tracing.bigbang.dev
```

* Remember to un-edit your hostfile when done