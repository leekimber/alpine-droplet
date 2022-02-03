# Digital Ocean Alpine Linux Image Generator

![Build Status](https://github.com/benpye/alpine-droplet/actions/workflows/build.yml/badge.svg?branch=master)

This is a tool to generate an Alpine Linux custom image for Digital Ocean. This ensures that the droplet will correctly configure networking and SSH on first boot using Digital Ocean's metadata service. To use this tool make sure you have `qemu-nbd`, `qemu-img`, `bzip2` and `e2fsprogs` installed. This will not work under the Windows Subsystem for Linux (WSL) as it mounts the image during generation.

Once these prerequisites are installed, clone the repo:
```
git clone https://github.com/benpye/alpine-droplet.git
```
Initialise the 'alpine-make-vim-image' submodule

```
cd alpine-droplet/
git submodule init
git submodule update
```

run:

```bash
# sudo ./build-image.sh
```

Note: Need root permission.

2022-02-03: After cloning, cd-ing into repo, I see:
```
$ sudo ./build-image.sh 
./build-image.sh: 13: ./build-image.sh: ./alpine-make-vm-image/alpine-make-vm-image: not found
```

We need the https://github.com/alpinelinux/alpine-make-vm-image repo as a submodule, so:

```
$ git submodule init
Submodule 'alpine-make-vm-image' (https://github.com/alpinelinux/alpine-make-vm-image.git) registered for path 'alpine-make-vm-image'

$ git submodule update
Cloning into 'alpine-make-vm-image'...
remote: Enumerating objects: 296, done.
remote: Counting objects: 100% (37/37), done.
remote: Compressing objects: 100% (26/26), done.
remote: Total 296 (delta 15), reused 28 (delta 9), pack-reused 259
Receiving objects: 100% (296/296), 62.33 KiB | 0 bytes/s, done.
Resolving deltas: 100% (167/167), done.
Checking connectivity... done.
Submodule path 'alpine-make-vm-image': checked out 'b0f0a05b1848e0ff363cfbb36bf319fc02132caa'
```

OK, let's try building again:

```
$ sudo ./build-image.sh 

> apk not found, downloading static apk-tools
ERROR: cannot verify gitlab.alpinelinux.org's certificate, issued by ‘CN=R3,O=Let's Encrypt,C=US’:
  Issued certificate has expired.
To connect to gitlab.alpinelinux.org insecurely, use `--no-check-certificate'.
```

Self-explanatory error. Workaround (not fix) is to modify alpine-make-vm-image's 'wgets()' function to ignore SSL cert errors:

```
vim alpine-make-vm-image/alpine-make-vm-image
```
On line 309, change:
```
&& wget -T 10 --no-verbose "$url" \
```
to
```
&& wget --no-check-certificate -T 10 --no-verbose "$url" \
```
And try again:

```
$ sudo ./build-image.sh 

...
muchas textas
...
```
Seems to have completed. Let's check:

```
$ ls -lh *.bz2
42M alpine-virt-image-2022-02-03-1802.qcow2.bz2
```

Which looks 42Mb closer to the description below:

This will produce `alpine-virt-image-{timestamp}.qcow2.bz2` which can then be uploaded to Digital Ocean and used to create your droplet. Check out their instructions at https://blog.digitalocean.com/custom-images/ for uploading the image and creating your droplet.

In this commit, the script will produce alpine `version 3.15` image. If you wanna build latest version, you can pull latest [alpine-make-vm-image repo](https://github.com/alpinelinux/alpine-make-vm-image): `git submodule foreach git pull origin master`

That git step completed silently but with no obvious difference on my system. Hence the manual intervention logged above.

Next step is to familiarise with [Digital Ocean's image upload procedure](https://docs.digitalocean.com/products/images/custom-images/how-to/upload/).

Which all worked fine for me. Thanks [Ben Pye](https://github.com/benpye/alpine-droplet), [jirutka](https://github.com/alpinelinux/alpine-make-vm-image/) and, of course, the [wonderful Alpine Linux team](https://alpinelinux.org/).

Oh, on that Digital Ocean droplet build, make sure you include your ssh keys and, if you are a long-time Digital Ocean customer, that your ssh keys are ed22519 or better.
