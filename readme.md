# Docker Study Notes

### Installation on New Linux (Debian)

1) **Install prerequisites:**

- `apt-transport-https`

- `ca-certificates`

- `curl`

- `gnupg-agent`

- `software-properties-common`

  _Note that some of these will already be available, so may just get updated._

```bash
$ sudo apt update -y
...
$ sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

2) **Pull Docker's public GPG key**

​		Get the GPG public key of the `Docker` `apt` repository and add it to our `apt-key` list.

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
```

​		Now if we wish we can verify the GPG key fingerprint.

```bash
$ sudo apt-key fingerprint
...
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
...
```

​		The `Docker` fingerprint id is the last two octets in the public key **`0EBFCD88`**.

3) **Add Docker repository**

​	Add the official `Docker` repository path to our trusted `apt` repositories.

```bash
$ sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) \
    stable"
```

4) **Install Docker engine**

​	Finally update `apt` and install the latest version of _Docker engine_ and _containerd_.

```bash
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io 
```

5) **Verify installation**

```bash
$ docker --version
```

​		This should display the version and build of `Docker` that got installed.

6) **Add your user to Docker group**

​	To access `Docker` as "non-root" user, add your user to the `docker` user group.

```bash
$ sudo usermod -aG docker $(whoami)
```

​		This should add the current user to the `docker` group. You might have to log out and log back in. After that you should be able to see yourself as part of the `docker` group if you try the following command.

```bash
$ groups
... ... docker
```

7) **Test Docker hello-world**

​	We can test our `Docker`  installation by trying to run a package called `hello-world` that `Docker` has published for this purpose. You should see something like below if we execute `docker run <repo-name>`

```bash
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:4cf9c47f86df71d48364001ede3a4fcd85ae80ce02ebad74156906caff5378bc
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

```

​		The `docker` `deamon` checked the local cache, and did not find an image(because it is the first time it is being run), it pulls it down from `Docker hub`, and then created a container instance and executed it.

_The official documentation for the installation is available here : https://docs.docker.com/engine/install/ubuntu/_

### Launching and Listing Containers

​		Another lightweight container we that is provided for us to explore is `alpine`. So we can try that.

```bash
$ docker run alpine
```

​		Now if we try listing the containers with `docker container ls` we will:

```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

```

​		Nothing, no containers. That is because the `image` was pulled down, and `container` instance `created` and `started`, but then it completes its command and exits with `container` in `stopped` state. If we want to see all containers regardless of their state we can add an additional switch to the command:

```bash
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
554d545422bd        alpine              "/bin/sh"           4 minutes ago       Exited (0) 4 minutes ago                        infallible_hawking
5d85da35abc2        hello-world         "/hello"            19 minutes ago      Exited (0) 19 minutes ago                       awesome_jones

```

### Launching in interactive mode

​		Containers that have a shell can be launched in `interactive` mode by using the `-i` or `--interactive` flag in the `run` command. Additionally the `-t` option should be provided to attach the `tty` to get a familiar interface to the shell. Whenever we launch a `container` interactively (or detached in the background) it is a good idea to give it a name with the `--name` flag.

```bash
$ docker run -it --name alpinec1 alpine
/ # hostname
84e4f60cb863
/ # whoami
root
/ # ls
bin    etc    lib    mnt    proc   run    srv    tmp    var
dev    home   media  opt    root   sbin   sys    usr

```

​		Now `alpine` (which is a very lightweight `linux` distro for containers) launches and attaches the shell. Then we are in as a `root` and we can interact with that isolated `container`.

​		If we check for all containers now, we should see:

```bash
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
84e4f60cb863        alpine              "/bin/sh"           50 seconds ago      Exited (0) 23 seconds ago                       alpinec1
554d545422bd        alpine              "/bin/sh"           28 minutes ago      Exited (0) 28 minutes ago                       infallible_hawking
5d85da35abc2        hello-world         "/hello"            43 minutes ago      Exited (0) 43 minutes ago                       awesome_jones

```

​		The first line is our interactive container that we had launched and exited from.

### Removing a container

​		We can remove containers from the list using the `rm` command.

```bash
$ docker rm alpine1
```

​		Now if we list all the containers it will no longer show up.

```bash
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
554d545422bd        alpine              "/bin/sh"           42 minutes ago      Exited (0) 42 minutes ago                       infallible_hawking
5d85da35abc2        hello-world         "/hello"            57 minutes ago      Exited (0) 57 minutes ago    
```

​		We can remove all the containers if we wish to, using a nested command to list all containers (or `docker ps`) and passing that to `doscker rm`.

```bash
$ docker rm $(docker ps -a -q)
OR
$ docker rm $(docker container -aq)
```

​		If we want to remove all 'stopped' containers we can do so using the `prune` command.

```bash
$ docker container prune
```

_Note: there are ways to format the output fro the `docker` commands using the `--format` flag. We can even get the output as `json` and use the `jq` utility (in `Linux`) to format/pretty-print the result._

```bash
$ docker container ls -a --format "{{json .}}" | jq '.'
```

### Running container as 'Detached'

​		If we wan the container to start and run in the background, we can run it in `detached` mode using the `-d` flag.

```bash
$ docker run -dt --name alp-1 alpine
0e07.......................................bc5faf840b61
```

​		This will launch a container of the `alpine` image in the background (_we need the `-`t flag as well for `tty` because otherwise the `alpine` instance will exit_).

​		`alipne` is a lightweight Linux distro for `Docker`. In fact we can check the OS details for the container with the following command (_from within the container shell_):

```bash
$ cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.12.0
PRETTY_NAME="Alpine Linux v3.12"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://bugs.alpinelinux.org/"
```

​		_Note that `uname` command will not work, as that will give the "host machine" OS!_

```bash
$ uname -a
Linux xxxxxxx 5.4.0-48-generic #52-Ubuntu SMP Thu Sep 10 10:58:49 UTC 2020 x86_64 Linux
```

​		We can now see the image with the normal `list` command.

```bash
$ docker container ls --format "{{json .}}" | jq '.'
{
  "Command": "\"/bin/sh\"",
  "CreatedAt": "2020-09-24 17:03:46 +0100 IST",
  "ID": "0e078283a83d",
  "Image": "alpine",
  "Labels": "",
  "LocalVolumes": "0",
  "Mounts": "",
  "Names": "alp-1",
  "Networks": "bridge",
  "Ports": "",
  "RunningFor": "About a minute ago",
  "Size": "0B",
  "Status": "Up About a minute"
}
```

​		_Note: the formatting part is just for better readability. It uses the command-line utility `jq` to format/pretty-print json._

​		Now we can stop the container with the `stop` command.

```bash
$ docker stop alp-1
```

​		And finally remove it using the `rm` command.

### Run 'temporary' containers 

​		A lot of the time we want to try out images and the containers to be "removed automatically" when it exits. To do this we can use the `--rm` flag with the `run` command.

```bash
$ docker run -dt --rm --name alp-1 alpine
```

​		Now when this container is stopped it will be removed as well.

### Specify 'restart' behavior

​		We can specify the restart behavior of the container using the `--restart` flag and give options such as `always`, `no`, `on-failure` etc.

### Starting a 'stopped' container

​		We can start a stopped container using the `start` command.

```bash
$ docker start alp-1
```

​		This will start the stopped container `alp-1` and we should be able to list it with the `container ls` command.

### Executing a command on a container

​		We can execute a command on a container from outside using the `exec` command and specifying the command to be executed.

```bash
$ docker exec alp-1 cat etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.12.0
PRETTY_NAME="Alpine Linux v3.12"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://bugs.alpinelinux.org/"
```

​		This executes the command `cat etc/os-release` within the container.

​		We can use `exec` to connect to our background `alpine` image, using similar flags (`-it`) like we used with `run`.

```bash
$ docker exec -it alp-1 ash
/ # 
```

​		This will execute the `alpine` shell `ash` and attach a `tty` to it so that we will be inside the container at the `#` prompt.

​		We can use this to try and install `nginx` onto our `alpine` container.

```bash
$ docker exec alp-1 apk add nginx
```

​		This executes `apk add nginx` on our container `alp-1` which will install `nginx` on `alpine`.

### Updating & copying files

​		We can use the `cp` command to copy files to and from containers.

```bash
$ docker cp alp-1:/etc/nginx/conf.d/default.conf .
```

​		This will copy the `default.conf` file from the path `/etc/nginx/conf.d/` in the container `alp-1` to our current working directory.

​		We can edit the file as needed and copy it back to the container with the same command by simply swapping the source and destination.

```bash
$ docker cp default.conf alp-1:/etc/nginx/conf.d/default.conf
```

### Renaming containers

​		If we check our current container with the `ls` command:

```bash
$ docker container ls --format '{{.Names}}\t{{.Status}}'
alp-1	Up 10 minutes
```

​		_Note: the `format` switch for getting name and status only._

​		Let us rename our container to say `web01` (we will be using this as web-server later on).

```bash
$ docker rename alp-1 web01
$ docker container ls --format '{{.Names}}\t{{.Status}}'
web01	Up 13 minutes
```

​		Now the `alp-1` container has been renamed to `web01`.

### View container metrics

​		Whilst in a production setup we may use a full scale monitoring setup such as 'Prometheus', in our lower environments we can use the `stats` command to do some spot check to see metrics about our containers.

```bash
$ docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
497bc1d8b05a        web01               0.00%               556KiB / 1.941GiB   0.03%               11.5kB / 0B         1.13MB / 0B         1
```

​		This command starts a display of the monitoring of our container/s (optionally specify a particular container if we need to). It provides metrics on memory usage, CPU usage, I/O etc/. To break out of this use the `^C`. We can make the display better by formatting it with the `--format` flag.

### Get container information

​		We can get the details of a running container using the `inspect` command.

```bash
$ docker inspect web01
```

​		This will give a lot of information about the container and its configuration, from storage mounts, network settings, to image details. For example if we wish to find the IP address details we could do the following:

```bash
$ docker inspect web01 | grep IP
```

### Create Image from container

​		We can use the `commit` command to create an image from a running container.

```bash
$ docker commit web01 alp-nginx-01
```

​		This will create an image from our current container which will have the configuration changes we have done on it. For example if we created a directory (such as `/run/nginx`), then the image will have that when we run a container from it.

​		If we now run a container (call it `web02`) from this new image, it will reflect the changes we had done to the container `web01` from which we 'committed' and created the image for this container.

```bash
$ docker run -dt --name web02 alp-nginx-01 
df66ea0..............................392a1209b5d49ce7ea05e
```

​		To make our docker container run the `nginx` server we installed on it and connect to it over port 80, we can now execute an `nginx` `daemon off` command within the container which will run `nginx` in the foreground. 

```bash
$ docker exec -dt web02 nginx -g 'daemon off;'
```

​		Note that we don't have to set the PID path to something like (`'pid /tmp/nginx.pid;'`) because this container will have a `/run/nginx/ngin.pid` path (which the default).

​		Now if `curl` to the IP address of the container (`web02`) we should see the content from the default page (`index.html`) served up. That is, if we have configured the `nginx` host configuration file accordingly. For example, we can see the way we configured it (again using the very handy `exec` command).

```bash
$ docker exec -t web02 cat /etc/nginx/conf.d/default.conf
server {
	listen 80 default_server;
	listen [::]:80 default_server;
        
        root /var/www;

        index index.html;
}
```

### Port mapping host_port : container_port

​		The next thing we might want to do would be to expose our `nginx` web server so that it can be accessed from outside the host. _Right now it runs on a local IP within the host_. To do this we can do a 'port-mapping' of the host to the container (using the `-p` flag). To see the open ports (tcp) listening on the container we can use `netstat`. 

```bash
$ docker exec -it web02 netstat -lt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       
tcp        0      0 0.0.0.0:http            0.0.0.0:*               LISTEN      
tcp        0      0 :::http                 :::*                    LISTEN      
```

​		To do port mapping we will have to re-run our container. To do that we have to first stop (_and remove if we want the same name_) the container. Next we do `docker run` with the `-p` flag from the same image.

```bash
$ docker run -dt --name web02 -p 80:80 alp-nginx-01
```

​		This will create a new instance of the container and it will have its port 80 mapped to the the host 80. Now if we again start the `nginx` it should work like before and we should be able to `curl` to the IP of the container.

```bash
$ docker exec -dt web02 nginx -g 'daemon off;'
$ curl 172.17.0.2
<html>
<head><title>Nginx Hello</title></head>
<body>
<center><h1>Hello from Nginx in Docker!</h1></center>
</body>
</html>
```

​		However now since the container's port 80 is mapped to the host port 80, we should be able to get the same page if we do a localhost (or the IP the host itself from outside).

```bash
$  curl localhost
<html>
<head><title>Nginx Hello</title></head>
<body>
<center><h1>Hello from Nginx in Docker!</h1></center>
</body>
</html>
```

### Image basics

​		An image is the "source" from which a running instance (container) is created. It specifies everything we need to run our application/service (such as the runtime, the libraries, configuration, environment variables, code). These are specified in 'layers'. These layers are cached because of which they are more reusable, quicker to deploy, and take up less space overall.

​		In the previous sections we played around with the containers that we launched from some images to configure it and setup the services that we want to run on it.  This is great for experimenting and learning, however in an actual scenario we would want to have everything setup and configured so that as soon as the container is run from the image it is up and ready to function. This is what we can use the `'Dockerfile'` for.

​		There is no need create and image from scratch. Most of the time an image is based on some **parent image** (like `nginx` or `node`). Images without a parent image is called a **base image**.

​		To get an initial understanding of `Dockerfile` we can look at the `hello-world` repository from Docker hub (`https://hub.docker.com/_/hello-world`). The `Dockerfile` will have the following content:

```dockerfile
FROM scratch
COPY hello /
CMD ["/hello"]
```

​		Each line in the `Dockerfile` is a "layer". Here the first line is `"scratch"` (which serves as template for **base images**) which means that this image has no **parent image** and is a **base image**.

​		For a **base image** the second line always has to specify the "root file system". Here it simply copies the `hello` binary to the root(`/`) directory.

​		Lastly it executes the command specified in the `CMD` instruction and when that is done it exists and the container is stopped.