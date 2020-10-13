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

​		Now since the container's port 80 is mapped to the host port 80, we should be able to get the same page if we do a localhost (or the IP the host itself from outside).

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

​		In the previous sections we played around with the containers that we launched from some already built images, configured them and setup the services that we want to run on it.  We also learned to `commit` containers to as images, so that we can create new images which persist the configuration changes we did. 

This is great for experimenting and learning, however in an actual scenario we would want to have everything setup and configured so that as soon as the container is run from the image it is up and ready to function. Also we normally want our containers to reflect the changes we make to our code. To do this we use the `'Dockerfile'`.

​		There is no need create and image from scratch. Most of the time an image is based on some **parent image** (like `nginx` or `node`). Images without a parent image is called a **base image**.

​		To get an initial understanding of `Dockerfile` we can look at the `hello-world` repository from Docker hub (`https://hub.docker.com/_/hello-world`). The `Dockerfile` will have the following content:

```dockerfile
FROM scratch
COPY hello /
CMD ["/hello"]
```

​		Each line in the `Dockerfile` is a "layer". Here the first line is `"scratch"` (which serves as template for **base images**, it can be thought of as the top of the image hierarchy) which means that this image has no **parent image** and is a **base image**.

​		For a **base image** the second line always has to specify the "root file system". Here it simply copies the `hello` binary to the root(`/`) directory.

​		Lastly it executes the command specified in the `CMD` instruction and when that is done it exists and the container is stopped.

#### "Imagize" a Node.js application

​		Now we will try to take a Node.js application (actually two applications, one 	web frontend, and another as its service backend), and package them as `Docker` images using `Dockerfile`, then we can run containers from these images.

​		We have two applications:

​		A web frontend `test-app`, with an HTML page and some JavaScript for browser action, and also to make HTTP request to a backend service application.

```bash
$ cd tech/node-study/test-app/
$ tree -L 2
.
├── index.html
├── index.js
├── package.json
├── package-lock.json
├── scripts
│   └── client.js
└── utils
│   └── call-apis.js
└── node_modules
    └── .....
```

​		A backend application `srv-app`, that provides the logic and exposes an HTTP service endpoint for the frontend application.

```bash
$  cd tech/node-study/srv-app/
$ tree -L 2
.
├── index.js
├── package.json
├── package-lock.json
└── utils
|   └── primes.js
└── node_modules
    └── .....
```

​		So the general approach for this would be to define a `Dockerfile` in each of the application directories, specify a parent image (`node: alpine` in this case), and copy our files across to some working directory (in the image), exclude the contents of `node_modules`, then specify commands to execute `npm install` and finally run the `node index.js` entry point. All this will be specified in each line to "Layer" the image creation.

​		Our `Dockerfile` for packaging a `Node.js` application normally looks like:

```dockerfile
# parent image is node:01-alpine
FROM node:10-alpine
# declare a variable for application path
ARG appPath=/home/node/app
# make dir for app path and set owner as node:node
RUN mkdir -p $appPath && chown -R node:node $appPath
# set app path as the current working dir here onwards
WORKDIR $appPath
# copy package.json & pcakage-lock.json to the app path
COPY package*.json ./
# install npm dependencies
RUN npm install
# copy rest of the app (src) files
COPY . .
# set user as 'node' here onwards
USER node
# expose port 3001 (our app listens on 3001)
EXPOSE 3001
# set the cmd to seceute as 'node index.js'
CMD ["node", "index.js"]
```

_Note: If we want to specify an `npm` `regsitry` (often when using an enterprise registry setup such as `artifactory`), we can declare that in the `Dockerfile`:_

```dockerfile
RUN npm config set registry http://registry.npmjs.org
```

​		Now we can build a docker image from the contents of this directory using this `Dockerfile`. To do that we use `docker build` command:

```bash
$ docker build -t "img-srv-app" .
```

​		This creates a docker image with the tag "img-srv-app:ltest", and we can see that with the `docker ls` command:

```bash
$ docker image ls --format "{{.Repository}}:{{.Tag}}"
img-srv-app:latest
node:10-alpine
hello-world:latest
```

​		Next we can launch a container from this image and access the `Node.js` application exposed endpoint.

```bash
$ docker run -dt --name srv-app img-srv-app
33341.............8fbb
```

​		If we want to see the result of the `run` action on the container we can use the `docker logs` command:

```bash
$ docker logs srv-app 
listening on port 3001
```

​		Now our application is listening on `port 3001`. To send a request to that we will need it's IP, which we can get from the `inspect` command (or use `exec` and run `ip a`).

```bash
$ docker inspect srv-app | grep 'IPAddress'
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
```

​		We can now try to make a request to our `srv-app` endpoint using this IP.

```bash
$ curl http://172.17.0.2:3001/apis/check-prime/23
{"error":false,"msg":"","is_prime":true}
```

​		If we do not want to discover the IP of the container try accessing with that, we can do a **port mapping** of the container exposed port to a port on the host (using the `-p` flag). This can only be done when we `run` the container, so if we want to apply to it a running container, we would have to stop it, remove it and run it again with the mapping:

```bash
$ docker run -dt --name srv-app -p 3001:3001 img-srv-app
003afb81................................................cdb2
```

​		Now we should be able to access the application endpoint via the `localhost`:

```bash
$ curl http://localhost:3001/apis/check-prime/23
{"error":false,"msg":"","is_prime":true}
```

#### Environment variables (ENV)

​		We have already seen above, how to use `ARG` to declare _"variables"_ which we can use within `Dockerfile`. The `ARG` variables are for use during the **image build** time (i.e. with the `docker build` command). We can use the `ENV` option to declare **environment variables** that can be specified when the **container is run**.

​		In our example if we want the ability top specify the **"port"** (that our application should listen on) when we run the container we can do it by modifying the `Dockerfile` as follows:

```dockerfile
...
# define an env variable 'PORT', give it a defualt value
ENV PORT=3001
# expoxe port using the 'port' env variable
EXPOSE $PORT
CMD ["node", "index.js"]
```

​		Now the identifier **"PORT"** will be available as an **environment variable** within the **Node.js** application, and we should be able to access it as:

```javascript
const port = process.env.PORT || 3001;
....
app.listen(port
    , () => console.log(`listening on port ${port}`));
```

​		The `ENV` variable set in the `Dockerfile` gets injected and made available as `process.env` attribute in the `Node.js` code.

​		This gives us the ability to specify the **port** that the application should listen on when we launch the container. If we wish to set it to `3002` for example, we could do that using the **`-e`** flag with the `run` command.

```bash
$ docker run -dt -e "PORT=3002" --name srv-app -p 3002:3002 img-srv-app
411aff.................................................6848
$ docker logs srv-app
listening on port 3002
```

​		Now we should be bale to make a request to **port** `3002`:

```bash
$  curl http://localhost:3002/apis/check-prime/23
{"error":false,"msg":"","is_prime":true}
```

​		We can extend this approach for our frontend application `test-app`, to use `ENV` variables to configure the backend service endpoint (from `srv-app`).

​		To do that we can add a couple of extra `ENV` variables in the `Dockerfile` for the `test-app`:

```dockerfile
...
ENV PORT=3000
ENV SRV_HOST=localhost
ENV SRV_PORT=3001
EXPOSE $PORT
CMD ["node", "index.js"]
```

​		We have two `ENV` variables - `SRV_HOST` and `SRV_PORT`. Now build the image from this file, and launch a container passing in the **host address** and **port** of the backend `srv-app` container. Before we can do that, we have to discover those values (using `docker inspect` and `docker exec` with `netstat`):

```bash
$ docker inspect srv-app | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
$ docker exec -it srv-app netstat -lt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       
tcp        0      0 :::3002                 :::*                    LISTEN
```

​		Now we know the values for **host** and **port**, we can provide them to the `test-app` container when we launch it (via the **`-e`** flags):

```bash
$ docker run --name test-app -dt -p 3000:3000 -e PORT=3000 -e SRV_HOST=172.17.0.2 -e SRV_PORT=3002 img-test-app
87a1cc06.............................................5197
$ docker logs test-app 
listening on port 3000
```

​		Now if we go to `http://localhost:3000` in a web browser, we should get our frontend application page loaded, and the when we try to execute some action this application should be able to talk to the backend application (`srv-app`) running in its container (provided the network settings allow it it, the default would be `bridge`).



-----------------

_Note: copying files from Linux machine to a raspberry pi:_

```bash
$ rsync -av -e ssh --exclude='node_modules/' . pi@192.168.xx.xx:~/test-app
```

- Use `rsync` utility (because with `scp` we cannot exclude files/directories easily)
- `-a` : recursively copy
- `v` : give verbose output
- `-e ssh` : encrypt channel with `ssh`
- `--exclude='node_modules/'` : exclude `node_modules` directory
- `.` : path to source (current directory in our case)
- `pi@192.168.xx.xx` : user@host
- `:~/test-app` : `:` path at destination (in raspberry pi)