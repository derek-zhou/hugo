---
title: Deploy a 32bit application to fly.io
slug: deploy_32bit_on_flyio
date: 2021-11-20
tags:
- web
description: fly.io gives you upto 3 256MB VMs in the free tier. 256MB is not a whole lot for a modern application; How to make the most use of it? You may want to deploy in 32 bit. 
---

[fly.io](https://fly.io) gives you upto 3 256MB VMs in the free tier; bigger VMs are also reasonably priced. 256MB is not a whole lot for a modern application; How do I make the most use of it? One way to save memory is to deploy in 32-bit. A 32-bit application can only address upto 4GB of ram (actually 3GB or even 2GB taking into account of the kernel and various other things). Obviously it is quite limited. However, if I only have 256 MB to begin with, then this limitation is a non-issue. Running in 32 bit, the application will use 4 bytes per pointer instead of 8 bytes per pointer, so for pointer heavy applications (any application written in a high level language is), the saving can be substantial.

Now, officially, fly.io does not support 32 bit Linux x86 images. However, we knows that: Linux KVM hypervisor runs 32 bit Linux guest unmodified, and AMD64 kernel runs 32bit userland applications unmodified, so this should be possible; it is only a matter of how. Here is how I did it.

## The big picture

To deploy in 32 bit, you need to build your application in 32 bit. Then, you need to deploy your application in a suitable docker image. I am most familiar with [Debian](https://Debian.org), so I want to deploy in a Debian Bullseye (11.x, latest stable release). And I want it to be 32 bit x86, instead of the usual 64 bit AMD64. I use the [Elixir](https://elixir-lang.org/) stack, which make everything easy. Still, some of the steps are not obvious.

## Build in 32 bit

The official [Phoenix deployment guide](https://hexdocs.pm/phoenix/releases.html#containers) suggests to build the application in a different Docker image prior to deployment. It will make CI easy because you can make use of Github actions or something like it. However, for better controllability, I have to build locally. The build artifact of a Phoenix application is something called an [OTP release](https://elixir-lang.org/getting-started/mix-otp/config-and-releases.html), which is a package including everything from the Erlang VM (BEAM) and up. It is portable so long as the system libraries such as libc and OpenSSL are the same between the build environment and the deployment enviroment. So we need to work backards from the intended deployment environment to setup our build environment.

The easiest way to get a 32bit environment locally is to use LXC (https://linuxcontainers.org/). Docker is great for deployment, but not as tweakable as an LXC container. If you have a local Debian AMD64 box, installing LXC and spin up a 32bit debian container is trival. Please read the `lxc-create` manpage for details.

Once you have a barebone image, you can just dive in and use `apt` to install most softwares you want: git, gcc, to name a few. Debian provides a lot of packages, except some of that are too old to be useful.

### Installing node.js

Debian's node package is quite out of date in the very fast paced Node.js world. Here is bummer No.1: In the infinite wistom of the Node.js maintainers, 32 bit Linux is no longer a supported architechture anymore. Luckily: they still provides unofficial 32bit build of nodejs from [here](https://unofficial-builds.nodejs.org/download/release/v16.13.0/) and that will do for me.

### Installing Erlang

Again, Debian's Erlang package is too old for my liking. 32bit Linux is still an officially supported arch for Erlang; so you can use a official binary package. But why not have some fun and compile one yourself? Erlang can be compiled with the normal ritual of `configure, make, make install` easily in your brand new environment; it should only take about 10 minutes.

### Installing Elixir

You don't want to use the Debian Elixir package. Elixir is even easier to compile than Erlang; it is built  on top of Erlang after all. Once you have Erlang up, it should only take 3 minutes to compile Elixir.

### Installing Docker

Debian even have one docker.io package. I don't know if it is compatible with whatever the latest toys want and I don't want to take any chance. Here come bummer #2: Docker does not support 32 bit build anymore an I cannot find a usable 32-bit binary package. Dealing with a Golang toolchain is outside of my comfort zone so I was stuck. Luckily, the way that the Go ecosystem works is that everyone distributes static linked binaries. Recall that our 32 bit environment is actually running on a 64 bit kernel, the [AMD64 binary](https://download.docker.com/linux/static/stable/), being statically linked, actually works like a charm!

### Installing flyctl

Of course flyctl does not support 32 bit build and Debian don't offer a package. However, we know that flyctl is a Golang program too; and with our previous insight, we can use use the AMD64 binary download from [here](https://github.com/superfly/flyctl/releases).


Now we have all the tools we need, we can build a standard OTP release for our Phoenix application. Then we move on to the next step, deployment.

## Deploy in 32 bit

In the 32-bit build environment, I can build OTP releases and Docker images to my liking. Docker recognizes this environment as a 64 bit environment, but you can provide command line switch to override it:

```
docker build --platform linux/386  .
```

And it should work with some warning printed. Here come aother bummer, the image cannot be deployed to fly.io! Flyctl complaints:

```
Step 9/12 : COPY --chown=nobody:root _build/prod/rel ./
Error error building: error rendering build status stream: failed to get destination image "sha256:25b4cabe2bbca2c88fe3d98e751789319c1c54281848836723db42c311d8929d":
image with reference sha256:25b4cabe2bbca2c88fe3d98e751789319c1c54281848836723db42c311d8929d
was found but does not match the specified platform: wanted linux/amd64, actual: linux/386

```

_What do we do now?_

The solution is actually very simple. Remember that an AMD64 kernel can run 32bit userland unmodified, we only need a docker template with a 64 bit kernel and a 32 bit userland to satisfy both flyctl and the 32 bit OTP release. And lo and behold, one such image is provided by [someone already](https://hub.docker.com/r/multiarch/debian-debootstrap)!

## Putting everything together

In summary:

1. I first built a fully functional 32 bit environment fully compatible with my intended deployment environment, using LXC on my AMD64 Linux pc.
1. I then install various tools using a number of ways, so I can do _everything_ including npm, mix, docker, flyctl, all inside the 32 bit environment.
1. Lastly, I use a multiarch docker image to house my locally built 32bit OTP release so that it can run on fly.io

Now my application consumes a lot less memory (from 24MB to 50MB+ depends on where you look) than before and is running happily on a puny little 256MB fly.io VM, with a lot of headroom to grow into.

By the way, the application I built is called [Gara (Get a room already!)](https://gara.fly.dev), a chatroom application. Give it a try, or [fork it at Github](https://github.com/derek-zhou/gara)!
