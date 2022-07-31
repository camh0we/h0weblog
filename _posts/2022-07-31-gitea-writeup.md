---
layout: post
title: Gitea
excerpt_separator: <!--more-->
tags: gitea homelab docker terraform
---

This is a step by step guide to spinning up a self hosted git server with docker and terraform.

<!--more-->

# goals

- ability to manage all of this from my laptop
- locally hosted git server
- ability to create and maintain local repositories
- ability to mirror from existing github repos
- ability to mirror to a github repo
- accessible via https with valid ssl
- single sign on oauth

# planning

This is the first big decision to tackle, and one I've gone round and round on many times. A few approaches I've tried in the past

- single `docker-compose.yml` file, everything hard coded
- single `docker-compose.yml` file, using extenion fields and yaml anchors
- multiple `docker-compose.yml` files, combine them as a build step
- `golang` templates and structs to define services in `go` and generate a single `docker-comose.yml`

Those all sorta worked, but got boring. Today I'm using the `terraform docker provider` to define and manage docker resources. This let's me work locally on my laptop, and when it's time to deploy or update things it's done via a `terraform plan` and `terraform apply`.

## environment setup

the hardware in play here

- mac -- osx -- m1 -- 16G
- hopper -- ubuntu -- x86 - 8G
- lenny -- ubuntu -- x86 -- 16G

### prerequisites

- `docker` installed and manageable by non-root user on all 3 hosts, all connected to the LAN, on the same subnet [docs](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)
  - if this is set up correctly i can run `docker ps` successfully on each machine without prefixing it with `sudo`
  - this is a little script to do the whole docker install thing, only tested onnfresh ubuntu 22.04 installs
- `terraform` installed and working on mac, this is the machine i physically work on [docs](https://learn.hashicorp.com/tutorials/terraform/install-cli)
  - if this is set up I can run `terraform -version` successfully

### bootstrap a terraform project

create a terraform directory with a `main.tf` file containing this:

```
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.13.0"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}
```

run these commands to initialize the terraforom project and apply the plan

```
❯ cd terraform
❯ terraform init

...

❯ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

 ...

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 7s [id=sha256:fd2d3e51789eafe943eec792c81e5297ae23570f2a24ed239118f3226e74a1ccnginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 0s [id=c13f98b7818f755ff06c4a30021176481ee8ffa139d870cb1d9435e3cf906bf5]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

❯ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
c13f98b7818f   fd2d3e51789e   "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   0.0.0.0:8000->80/tcp   tutorial
```

check out the page in a browser

[![Screen Shot 2022-07-30 at 8.28.41 PM.png](https://bookstack.shrevelab.io/uploads/images/gallery/2022-07/scaled-1680-/screen-shot-2022-07-30-at-8-28-41-pm.png)](https://bookstack.shrevelab.io/uploads/images/gallery/2022-07/screen-shot-2022-07-30-at-8-28-41-pm.png)

okay, so we can use terraform to spin up a local container, lets get it running on the other machine.

### expose the docker daemon over a tcp port

```
sudo systemctl edit docker.service

# paste into file
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376

sudo systemctl daemon-reload
sudo systemctl restart docker.service
```

check it worked with

```
> ps -aux | grep dockerd
root       15997  0.9  1.0 1679280 80168 ?       Ssl  20:44   0:00 /usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376
```

destroy what we have locally

```
terraform plan -destroy -out tfplan
terraform apply "tfplan"
```

update the provider block in the terraform to point at the new docker daemon

```
provider "docker" {
	host = "http://10.0.0.110:2376"
}
```

then plan and apply the terraform

```
terraform plan -out tfplan
terraform apply "tfplan"
```

navigate to `10.0.0.110:8000` in a web browser to see the same nginx starting page

# locally hosted git server

[gitea](https://gitea.io/en-us/)
