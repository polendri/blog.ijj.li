+++
title = "Self-hosting Secure Services Using Podman, Part 1"
date = "2020-04-13T00:00:00-04:00"
categories = ["selfhosting"]
tags = []
draft = true
+++

_This post is Part 1 in a series._

In 2020, I've been on a quest to build the perfect infrastructure for self-hosting services, both private ones for my
LAN and public hobby projects. The goals I've been looking to achieve with my setup are simple:

- **Automated**: It should take a single command to provision a system, upgrade a system, and start or stop a service.
- **Reproducible**: If I destroy a VPS instance or physically fry a home computer, it should be trivial to start over
  on a fresh system.
- **Secure**: It should follow every security best practice it possibly can.

I've settled on three tools for building this infrastructure, [Terraform](https://www.terraform.io/),
[Ansible](https://www.ansible.com/), and [Podman](https://podman.io/).

## Managing cloud infrastructure with Terraform

Terraform is a tool that enables you to declaratively define your cloud infrastructure and then create and destroy it
with a simple command. For example, this blog runs on a DigitalOcean VPS instance configured like this:

```
provider "digitalocean" {
  token = var.digitalocean_token
}

resource "digitalocean_droplet" "ijj_li" {
  image    = "ubuntu-19-10-x64"
  name     = "ijj.li"
  region   = "tor1"
  size     = "s-1vcpu-1gb"
  ssh_keys = var.digitalocean_ssh_keys
}

resource "digitalocean_floating_ip_assignment" "ijj_li" {
  ip_address = var.floating_ip
  droplet_id = digitalocean_droplet.ijj_li.id
}
```

What this says is:

- I want a 1GB Droplet created, running the Ubuntu 19.10 image.
- I want an existing floating IP (which I point my `ijj.li` DNS entry to so that it's easy to swap between hosts)
  mapped to this newly-created droplet.

Teraform can create that infrastructure, and when you make modifications later on, it's able to intelligently decide
what needs to change in order for things to match your new configuration.

## Ansible for administering hosts

Ansible is an extremely generic tool for "doing things on remote hosts", which it accomplishes via SSH. It can run
arbitrary commands, but also has modules to greatly simplify common operations like installing packages and manipulating
files.

A task for creating a user on the remote host looks like this:

```
- name: "Create 'srv' user for running services"
    user:
      name: "srv"
      group: "users"
```

This invokes the "user" module with some parameters. Tasks run in "playbooks" (essentially sequences of tasks), so a
playbook can contain a sequence of tasks which fully configure a system.

What makes this especially magical is the way that Ansible tasks are designed to be idempotent: if you were to run the
above "Create user" task a second time, you would find that it still runs successfully. The only difference is that
Ansible's output, instead of saying "changed", would say "ok" to indicate that the host's state already matches what
the task declares. This makes Ansible tasks somewhat declarative as well: you tasks can define the way you want a host
to be set up, and you can freely re-run your playbooks as you develop them.

I had stayed away from Ansible for several years because I disliked the way that it isn't truly declarative. If you
write a task to modify a config file and then later on you remove that task, nothing will undo that modification for
you; it's on you to clean that up. Fully declarative systems like [NixOS](https://nixos.org/) (where the state of your
entire host is defined in a single config file) and [Docker](https://www.docker.com/) container images (which start from
a clean base image every time) provide complete assurance that there's no untracked configuration, so Ansible is a bit
of a dirty compromise to me.

A dirty compromise it may be, but I think what I needed to accept sooner was that for self-hosting purposes, you just
can't avoid having regular old stateful hosts. The bottom line is that all sorts of devices on my home network can't
host Docker images or run NixOS: my printserver on a Raspberry Pi Zero for example, or my router.

But _everything_ runs SSH, so everything can be administered with Ansible so long as my admin machine has its SSH key
installed on the host. That means Ansible can be a single, centralized place for administering my entire network.
Very handy!

## Podman for managing containerized services

Podman is a Docker alternative developed by Red Hat; essentially it's less mature and rougher around the edges, but
makes some different and advantageous architectural choices compared to Docker.

The key difference between Podman and Docker is that Podman is daemonless: unlike Docker, there's no daemon process
running as root which manages containers.

This has a number of implications:

- Any daemon running as root is a security liability, since if it were to be compromised, an attacker can leverage it
  for root access of the system. This has been leveraged in the past with the Docker daemon. No daemon means a smaller
  attack surface.
- Since there's no daemon to do its own container lifecycle management, using Podman is dependent on using the host's
  service manager (e.g. systemd) for starting, stopping and restarting containers. This a simplification of the overall
  system (since the existing service manager is being used instead of adding another manager to do the same thing), but
  since systemd isn't tightly integrated with Podman, it's overall a bit more work to set up.
- This makes it possible to run Podman as an unprivileged user, something I make use of extensively.
- There's no tool like Docker's [Ouroboros](https://github.com/pyouroboros/ouroboros) to automatically keep container
  images up-to-date, since Ouroboros relies on accessing the Docker daemon to start/restart containers. For Podman,
  we would need to do this as a non-containerized service on the host.

I like the "technically correct" design choices of Podman compared to Docker, but I had a tough time choosing between
the two because the immaturity of Podman as a project really does have an impact on using it (mainly just how little
documentation, blogs, and StackOverflow answers that exist out there for it). Docker even has a
[rootless mode](https://docs.docker.com/engine/security/rootless/) these days, negating one of the key advantages of
Podman.

What it came down to was one blocker with Docker: host volume mounts can't have their permissions set for use by a
non-root user inside the container. I'll get into it in later posts, but running as a non-root user inside containers is
an important security practice for me, so I need to be able to set the permissions of mounted volumes such that the user
in the container can access them. With Podman, I can run `podman unshare chown -R 1000:100 mydir` on the host, to set
its permissions so that the container user with UID 1000 can access the `mydir` volume.

The purpose of using containers is twofold for me:

1. Provide a way to cleanly install and run services without gumming up the host system over time (with installed
   dependencies, leftover data/config files, etc)
2. Securely isolate services from the host system so that the host system is safe if they're compromised.

## What's next

With these three tools, we can build a super-secure, easy-to-administer foundation for self-hosting. In the next post,
I'll get into those security practices and how they guide the design and configuration of my hosts.

The config for my infrastructure is available for viewing at https://github.com/polendri/infrastructure, feel free to
take a look!
