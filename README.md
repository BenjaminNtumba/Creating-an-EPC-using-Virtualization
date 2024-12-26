# Install Terraform and Docker On Linux
This documentation details the installation of the tools required to manage future 5G servers on an Ubuntu/Debian or Windows system. Although other Linux distributions (CentOS/RHEL, Fedora, Amazon Linux) can be used, Ubuntu is highly recommended due to its large community, rich repositories and compatibility with many network devices. These factors make Ubuntu the reference distribution for network environments.
I would advise against installation on a virtual machine, rather on the main operating system, unless the virtual machine supports `kvm_intel` or `kvm_amd64` to run Docker and support virtualization inside a virtual machine.

# Installing Terraform on Debian

Make sure your system is up to date and that you've installed the `gnupg`, `software-properties-common` and `curl` packages. You'll use these packages to verify the HashiCorp GPG signature and install the HashiCorp Debian package repository. 

```c
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```

Install HashiCorp's `GPG key`

```c
wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

```

Check key fingerprint

```c
gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint

```

Add the official HashiCorp repository to your system. The `lsb_release` -cs command will indicate the code name of the distribution version currently used by your system, such as buster, groovy or sid.

```c
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

```

Download HashiCorp packaging information.

```c
sudo apt update
```

Install Terraform from the new repository

```c
sudo apt-get install terraform
```

# Installing Docker on Debian

Operating system requirements
To install Docker Engine, you need a `64-bit version` of one of the following `Debian` distributions:
Debian Bookworm 12 (stable)
Debian Bullseye 11 (old stable)
Docker Engine for Debian is compatible with `x86_64` (or `amd64`), `armhf`, `arm64` and `ppc64le` (`ppc64el`) architectures.

Uninstalling older versions
Before installing the Docker Engine, you need to uninstall any conflicting packages.
Your Linux distribution may provide unofficial Docker packages that may conflict with the official packages supplied by Docker. You must uninstall these packages before installing the official version of Docker Engine.
The unofficial packages to be uninstalled are as follows

- [docker.io](http://docker.io/)
- docker-compose
- docker-doc
- podman-docker

Run the following command to `uninstall the conflicting` packages 

```c
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

Download gnome-terminal

```c
sudo apt install gnome-terminal
```

Configure the Docker apt repository.

```c
# Add the official Docker GPG key:
 sudo apt-get update
 sudo apt-get install ca-certificates curl
 sudo install -m 0755 -d /etc/apt/keyrings
 sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
 sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add deposit to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

```

> Note!: If you use a derivative distribution, such as Kali Linux, you may need to substitute the part of this command that's expected to print the version codename:
> 

```bash
(. /etc/os-release && echo "$VERSION_CODENAME")
```

> Replace this part with the codename of the corresponding Debian release, such as `bookworm`.(this is an example, confirm your Debians codename)
> 

 Install the latest version of Docker packages

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

verify successful installation by running the hello-world image 

```bash
sudo systemctl start docker
sudo docker run hello-world
```

Download the latest [[DEB](https://desktop.docker.com/linux/main/amd64/docker-desktop-amd64.deb?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-linux-amd64)](https://desktop.docker.com/linux/main/amd64/docker-desktop-amd64.deb?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-linux-amd64) package. For checksums, see the release notes.

Install the package using apt as follows

```bash
 sudo apt-get update
 cd Downloads
 sudo apt-get install ./docker-desktop-amd64.deb

```

If  user is a Bash user

```bash
touch ~/.bashrc
```

if is  user a Zshrc user

```bash
touch ~/.zshrc
```

Then Install the Auto complete package

```bash
terraform -install-autocomplete
```

Create a directory called learn-terraform-docker-container.

```bash
mkdir learn-terraform-docker-container
```

Navigate in the working directory

```bash
cd learn-terraform-docker-container
```

In the working directory, create a file called [[main.tf](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-build)](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-build) and paste the following Terraform configuration into it. 

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "tutorial"

  ports {
    internal = 80
    external = 8000
  }
}

```

Initialize the project, which downloads a plugin called provider that allows Terraform to interact with Docker.

```bash
terraform init
```

Provision the NGINX server container with Apply. When Terraform asks you to confirm, type yes and press ENTER.

```bash
sudo terraform apply
```

Verify the existence of the NGINX container by visiting [[localhost:8000](http://localhost:8000/)](http://localhost:8000/) in your web browser or running `docker ps` to see the container.


