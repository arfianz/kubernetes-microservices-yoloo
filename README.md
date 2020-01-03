# Deploy microservices on Kubernetes

Kubernetes (AKA k8s) has gained widespread adoption in recent years as a platform for microservices due to its ability to seamlessly automate app deployment at scale. Pinterest uses a suite of over 1000 microservices to power their “discovery engine”. Imagine having to configure and manage servers to run these services manually. It’s an Engineer’s nightmare to say the least.

Kubernetes bills itself as “a portable, extensible open-source platform for managing containerized workloads and services”. In simple terms, Kubernetes helps to automate the deployment and management of containerized applications. This means we can package an app (code, dependencies and config) in a container and hand it over to Kubernetes to deploy and scale without worrying about our infrastructure. Under the hood, Kubernetes decides where to run what, monitors the systems and fixes things if something goes wrong.

## Microservice architecture

I particularly like [James Lewis’ and Martin Fowler’s definition of microservices](https://martinfowler.com/microservices/) as it points out why Kubernetes is such a good solution for the architecture. “The microservice architectural style is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by **fully automated deployment machinery**. There is a bare minimum of centralized management of these services, which may be written in different programming languages and use different data storage technologies”. Kubernetes is in fact a “fully automated deployment machine” that provides a powerful abstraction layer atop server infrastructure.

## Common Kubernetes terms

Below are some common terms associated with Kubernetes.

- **Node**: A node is a single machine in a Kubernetes cluster. It can be a virtual or physical machine.
- **Cluster**: A cluster consists of at least one master machine and multiple worker machines called nodes.
- **Pod**: A pod is the basic unit of computing in Kubernetes. Containers are not run directly. Instead, they are wrapped in a pod.
- **Deployment**: A deployment is used to manage a pod or set of pods. Pods are typically not created or managed directly. Deployments can automatically spin up any number of pods. If a pod dies, a deployment can automatically recreate it as well.
- **Service**: Pods are mortal. Consumers should however not be burdened with figuring out what pods are available and how to access them. Services keep track of all available pods of a certain type and provide a way to access them.

This repo contains the source code for my tutorial on [deploying microservices on Kubernetes](http://alphacoder.xyz/deploy-microservices-on-kubernetes). It houses two microservices: _detector_ and _viewer_. The detector service is a Python/Flask app that uses a pre-trained [YOLO](https://www.youtube.com/watch?v=Cgxsv1riJhI) v3 model to detect [common objects](detector/classes.txt) like laptops and chairs in an image. It returns a link to an output image of object predictions and a list of the detected objects. The viewer service is a PHP app which provides a front-end for uploading new images and viewing the images processed by the detector service.

## Install PHP and Composer

[Composer](https://getcomposer.org/) is a popular dependency management tool for PHP, created mainly to facilitate installation and updates for project dependencies. It will check which other packages a specific project depends on and install them for you, using the appropriate versions according to the project requirements.

### Install PHP

Before you download and install Composer, you’ll want to make sure your server has all dependencies installed.

First, update the package manager cache by running:

```bash
$ sudo apt update
```

Now, let’s install the dependencies. We’ll need curl in order to download Composer and php-cli for installing and running it. The php-mbstring package is necessary to provide functions for a library we’ll be using. git is used by Composer for downloading project dependencies, and unzip for extracting zipped packages. Everything can be installed with the following command:

```bash
$ sudo apt install curl php-cli php-mbstring git unzip php-curl
```

With the prerequisites installed, we can install Composer itself.

### Install Composer

Composer provides an [installer](https://getcomposer.org/installer), written in PHP. We’ll download it, verify that it’s not corrupted, and then use it to install Composer.

Make sure you’re in your home directory, then retrieve the installer using curl:

```bash
$ cd ~
$ curl -sS https://getcomposer.org/installer -o composer-setup.php
```

Next, verify that the installer matches the SHA-384 hash for the latest installer found on the [Composer Public Keys / Signatures](https://composer.github.io/pubkeys.html) page. Copy the hash from that page and store it as a shell variable:

```bash
$ HASH=baf1608c33254d00611ac1705c1d9958c817a1a33bce370c0595974b342601bd80b92a3f46067da89e3b06bff421f182
```

Make sure that you substitute the latest hash for the highlighted value.

Now execute the following PHP script to verify that the installation script is safe to run:

```bash
$ php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
Installer verified
```

If you see Installer corrupt, then you’ll need to redownload the installation script again and double check that you’re using the correct hash. Then run the command to verify the installer again. Once you have a verified installer, you can continue.

To install composer globally, use the following command which will download and install Composer as a system-wide command named composer, under /usr/local/bin:

```bash
$ sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
All settings correct for using Composer
Downloading...

Composer (version 1.9.1) successfully installed to: /usr/local/bin/composer
Use it: php /usr/local/bin/composer
```

To test your installation, run:

```bash
$ composer
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 1.9.1 2019-11-01 17:20:17

Usage:
  command [options] [arguments]

Options:
  -h, --help                     Display this help message
  -q, --quiet                    Do not output any message
  -V, --version                  Display this application version
      --ansi                     Force ANSI output
      --no-ansi                  Disable ANSI output
  -n, --no-interaction           Do not ask any interactive question
      --profile                  Display timing and memory usage information
      --no-plugins               Whether to disable plugins.
  -d, --working-dir=WORKING-DIR  If specified, use the given directory as working directory.
      --no-cache                 Prevent use of the cache
  -v|vv|vvv, --verbose           Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Available commands:
  about                Shows the short information about Composer.
  archive              Creates an archive of this composer package.
  browse               Opens the package's repository URL or homepage in your browser.
  check-platform-reqs  Check that platform requirements are satisfied.
  clear-cache          Clears composer's internal package cache.
  clearcache           Clears composer's internal package cache.
  config               Sets config options.
  create-project       Creates new project from a package into given directory.
  depends              Shows which packages cause the given package to be installed.
  diagnose             Diagnoses the system to identify common errors.
  dump-autoload        Dumps the autoloader.
  dumpautoload         Dumps the autoloader.
  exec                 Executes a vendored binary/script.
  global               Allows running commands in the global composer dir ($COMPOSER_HOME).
  help                 Displays help for a command
  home                 Opens the package's repository URL or homepage in your browser.
  i                    Installs the project dependencies from the composer.lock file if present, or falls back on the composer.json.
  info                 Shows information about packages.
  init                 Creates a basic composer.json file in current directory.
  install              Installs the project dependencies from the composer.lock file if present, or falls back on the composer.json.
  licenses             Shows information about licenses of dependencies.
  list                 Lists commands
  outdated             Shows a list of installed packages that have updates available, including their latest version.
  prohibits            Shows which packages prevent the given package from being installed.
  remove               Removes a package from the require or require-dev.
  require              Adds required packages to your composer.json and installs them.
  run                  Runs the scripts defined in composer.json.
  run-script           Runs the scripts defined in composer.json.
  search               Searches for packages.
  self-update          Updates composer.phar to the latest version.
  selfupdate           Updates composer.phar to the latest version.
  show                 Shows information about packages.
  status               Shows a list of locally modified packages, for packages installed from source.
  suggests             Shows package suggestions.
  u                    Upgrades your dependencies to the latest version according to composer.json, and updates the composer.lock file.
  update               Upgrades your dependencies to the latest version according to composer.json, and updates the composer.lock file.
  upgrade              Upgrades your dependencies to the latest version according to composer.json, and updates the composer.lock file.
  validate             Validates a composer.json and composer.lock.
  why                  Shows which packages cause the given package to be installed.
  why-not              Shows which packages prevent the given package from being installed.
```

## Deploying on Minikube

### Start Minikube

Start up the Kubernetes cluster with Minikube, giving it some extra resources.

```bash
$ minikube start --memory 8000 --cpus 4 --vm-driver=kvm2
😄  minikube v1.6.2 on Ubuntu 18.04
✨  Selecting 'kvm2' driver from user configuration (alternates: [virtualbox none])
🔥  Creating kvm2 VM (CPUs=4, Memory=8000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.17.0 on Docker '19.03.5' ...
🚜  Pulling images ...
🚀  Launching Kubernetes ... 
⌛  Waiting for cluster to come online ...
🏄  Done! kubectl is now configured to use "minikube"
```

### View the Dashboard

View the Minikube Dashboard, a web UI for managing deployments.

```bash
$ minikube dashboard url
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:34457/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

![dash-kube](./kube-dash.png?raw=true)

### Setup Private Registry

Set up the cluster registry by applying a .yaml manifest file.

```bash
$ kubectl apply -f manifests/registry.yaml
persistentvolume/registry created
persistentvolumeclaim/registry-claim created
service/registry created
service/registry-ui created
deployment.apps/registry created
```

Wait for the registry to finish deploying using the following command. Note that this may take several minutes.

```bash
$ kubectl rollout status deployments/registry
deployment "registry" successfully rolled out
```

View the registry user interface in a web browser.

```bash
$ minikube service registry-ui
|-----------|-------------|-------------|----------------------------|
| NAMESPACE |    NAME     | TARGET PORT |            URL             |
|-----------|-------------|-------------|----------------------------|
| default   | registry-ui | registry    | http://192.168.39.77:30319 |
|-----------|-------------|-------------|----------------------------|
🎉  Opening service default/registry-ui in default browser...
```

![registry-ui](./registry-ui.png?raw=true)

We’ve built the image, but before we can push it to the registry, we need to set up a temporary proxy. By default the Docker client can only push to HTTP (not HTTPS) via localhost. To work around this, we’ll set up a Docker container that listens on 127.0.0.1:30400 and forwards to our cluster. First, build the image for our proxy container.

```bash
$ docker build -t socat-registry -f socat/Dockerfile socat
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM alpine:latest
latest: Pulling from library/alpine
e6b0cf9c0882: Pull complete 
Digest: sha256:2171658620155679240babee0a7714f6509fae66898db422ad803b951257db78
Status: Downloaded newer image for alpine:latest
 ---> cc0abc535e36
Step 2/4 : RUN apk update && apk upgrade && apk add bash socat
 ---> Running in b7b9c7078e92
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/community/x86_64/APKINDEX.tar.gz
v3.11.2-25-g58afcd742e [http://dl-cdn.alpinelinux.org/alpine/v3.11/main]
v3.11.2-24-g7cfe3a1534 [http://dl-cdn.alpinelinux.org/alpine/v3.11/community]
OK: 11261 distinct packages available
(1/2) Upgrading libcrypto1.1 (1.1.1d-r2 -> 1.1.1d-r3)
(2/2) Upgrading libssl1.1 (1.1.1d-r2 -> 1.1.1d-r3)
OK: 6 MiB in 14 packages
(1/6) Installing ncurses-terminfo-base (6.1_p20191130-r0)
(2/6) Installing ncurses-terminfo (6.1_p20191130-r0)
(3/6) Installing ncurses-libs (6.1_p20191130-r0)
(4/6) Installing readline (8.0.1-r0)
(5/6) Installing bash (5.0.11-r1)
Executing bash-5.0.11-r1.post-install
(6/6) Installing socat (1.7.3.3-r1)
Executing busybox-1.31.1-r8.trigger
OK: 15 MiB in 20 packages
Removing intermediate container b7b9c7078e92
 ---> e39d43db0d95
Step 3/4 : COPY entrypoint.sh entrypoint.sh
 ---> 9186ccc7e39a
Step 4/4 : ENTRYPOINT ["sh", "./entrypoint.sh"]
 ---> Running in 001b64fb5fa5
Removing intermediate container 001b64fb5fa5
 ---> 6f1cfd3f6ada
Successfully built 6f1cfd3f6ada
Successfully tagged socat-registry:latest
```

Now run the proxy container from the newly created image. (Note that you may see some errors; this is normal as the commands are first making sure there are no previous instances running.)

```bash
$ docker run -d -e "REG_IP=`minikube ip`" -e "REG_PORT=30400" --name socat-registry -p 30400:5000 socat-registry
5064631db9ba8d94f4c9e6d823f5eafa76a5dd02282360cd8ef9734ddebedfb5
```

Check the container

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                     NAMES
5064631db9ba        socat-registry      "sh ./entrypoint.sh"   4 seconds ago       Up 1 second         0.0.0.0:30400->5000/tcp   socat-registry
```

With our proxy container up and running, we can now push our application images to the local repository.


## Deploy Yolo

In this tutorial, we’ll be deploying a simple microservices app called Yoloo. Yoloo uses a pre-trained YOLO ([You Only Look Once](https://www.youtube.com/watch?v=Cgxsv1riJhI)) model to detect common objects such as bottles and humans in an image. It comprises two microservices, detector and viewer. The detector service is a Python/Flask app which takes an image and passes it through the YOLO model to identify the objects in it. The viewer service is a PHP app that acts as a front-end by providing a User Interface for uploading and viewing the images. The app is built to use two external, managed services: Cloudinary for image hosting and Redis for data storage.

Fist, we download the YOLO weights in the detector directory.

```bash
$ cd detector
$ wget https://pjreddie.com/media/files/yolov3.weights
--2020-01-02 17:35:03--  https://pjreddie.com/media/files/yolov3.weights
Resolving pjreddie.com (pjreddie.com)... 128.208.4.108
Connecting to pjreddie.com (pjreddie.com)|128.208.4.108|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 248007048 (237M) [application/octet-stream]
Saving to: ‘yolov3.weights’

yolov3.weights           100%[=================================>] 236,52M   317KB/s    in 7m 32s  

2020-01-02 17:42:37 (535 KB/s) - ‘yolov3.weights’ saved [248007048/248007048]
```

Change directory to viewer/ and install the PHP dependencies. You need to have PHP and Composer installed.

```bash
$ cd ../viewer
$ composer install
Loading composer repositories with package information
Installing dependencies (including require-dev) from lock file
Package operations: 5 installs, 0 updates, 0 removals
  - Installing cloudinary/cloudinary_php (1.13.0): Downloading (100%)         
  - Installing predis/predis (v1.1.1): Downloading (100%)         
  - Installing symfony/polyfill-ctype (v1.10.0): Downloading (100%)         
  - Installing phpoption/phpoption (1.5.0): Downloading (100%)         
  - Installing vlucas/phpdotenv (v3.3.0): Downloading (100%)         
predis/predis suggests installing ext-phpiredis (Allows faster serialization and deserialization of the Redis protocol)
Generating autoload files
```

### Build Images

Build the docker images for the microservices

#### Detector Image

```bash
$ docker build -t 127.0.0.1:30400/detector-svc -f detector/Dockerfile detector
Sending build context to Docker daemon    248MB
Step 1/9 : FROM python:3.6-stretch
3.6-stretch: Pulling from library/python
146bd6a88618: Already exists 
9935d0c62ace: Already exists 
db0efb86e806: Already exists 
e705a4c4fd31: Already exists 
c877b722db6f: Already exists 
5d026340ca86: Pull complete 
4efc9add42bb: Pull complete 
d765956ebf6f: Pull complete 
0c84e713f381: Pull complete 
Digest: sha256:6c2a01a0dbec322d65e1cbb452649217d955ea53fabe45f98b49a96359868bf9
Status: Downloaded newer image for python:3.6-stretch
 ---> 21f01d5542ca
Step 2/9 : EXPOSE 8080
 ---> Running in 6fbfc623aa00
Removing intermediate container 6fbfc623aa00
 ---> 3e5c28f13225
Step 3/9 : RUN mkdir /www
 ---> Running in 1f513465b45f
Removing intermediate container 1f513465b45f
 ---> 2f1316a40501
Step 4/9 : WORKDIR /www
 ---> Running in 4acd120e2302
Removing intermediate container 4acd120e2302
 ---> 0b907c1c9542
Step 5/9 : COPY requirements.txt /www/
 ---> 2b098113f208
Step 6/9 : RUN pip install -r requirements.txt
 ---> Running in e68bc614dc51
Collecting flask==1.0.2
  Downloading https://files.pythonhosted.org/packages/7f/e7/08578774ed4536d3242b14dacb4696386634607af824ea997202cd0edb4b/Flask-1.0.2-py2.py3-none-any.whl (91kB)
Collecting python-dotenv==0.8.2
  Downloading https://files.pythonhosted.org/packages/85/9f/b76a51bb851fa25f7a162a16297f4473c67ec42dd55e4f7fc5b43913a606/python_dotenv-0.8.2-py2.py3-none-any.whl
Collecting gunicorn==19.8.1
  Downloading https://files.pythonhosted.org/packages/55/cb/09fe80bddf30be86abfc06ccb1154f97d6c64bb87111de066a5fc9ccb937/gunicorn-19.8.1-py2.py3-none-any.whl (112kB)
Collecting opencv-python==4.0.0.21
  Downloading https://files.pythonhosted.org/packages/37/49/874d119948a5a084a7ebe98308214098ef3471d76ab74200f9800efeef15/opencv_python-4.0.0.21-cp36-cp36m-manylinux1_x86_64.whl (25.4MB)
Collecting numpy==1.16.0
  Downloading https://files.pythonhosted.org/packages/7b/74/54c5f9bb9bd4dae27a61ec1b39076a39d359b3fb7ba15da79ef23858a9d8/numpy-1.16.0-cp36-cp36m-manylinux1_x86_64.whl (17.3MB)
Collecting cloudinary==1.15.0
  Downloading https://files.pythonhosted.org/packages/b6/8a/b7b83e647451273fc075d203b56c4d74f2315ebe13972ddda526c1683c0e/cloudinary-1.15.0.tar.gz (143kB)
Collecting Werkzeug>=0.14
  Downloading https://files.pythonhosted.org/packages/ce/42/3aeda98f96e85fd26180534d36570e4d18108d62ae36f87694b476b83d6f/Werkzeug-0.16.0-py2.py3-none-any.whl (327kB)
Collecting itsdangerous>=0.24
  Downloading https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting click>=5.1
  Downloading https://files.pythonhosted.org/packages/fa/37/45185cb5abbc30d7257104c434fe0b07e5a195a6847506c074527aa599ec/Click-7.0-py2.py3-none-any.whl (81kB)
Collecting Jinja2>=2.10
  Downloading https://files.pythonhosted.org/packages/65/e0/eb35e762802015cab1ccee04e8a277b03f1d8e53da3ec3106882ec42558b/Jinja2-2.10.3-py2.py3-none-any.whl (125kB)
Collecting six
  Downloading https://files.pythonhosted.org/packages/65/26/32b8464df2a97e6dd1b656ed26b2c194606c16fe163c695a992b36c11cdf/six-1.13.0-py2.py3-none-any.whl
Collecting mock
  Downloading https://files.pythonhosted.org/packages/05/d2/f94e68be6b17f46d2c353564da56e6fb89ef09faeeff3313a046cb810ca9/mock-3.0.5-py2.py3-none-any.whl
Collecting urllib3
  Downloading https://files.pythonhosted.org/packages/b4/40/a9837291310ee1ccc242ceb6ebfd9eb21539649f193a7c8c86ba15b98539/urllib3-1.25.7-py2.py3-none-any.whl (125kB)
Collecting certifi
  Downloading https://files.pythonhosted.org/packages/b9/63/df50cac98ea0d5b006c55a399c3bf1db9da7b5a24de7890bc9cfd5dd9e99/certifi-2019.11.28-py2.py3-none-any.whl (156kB)
Collecting MarkupSafe>=0.23
  Downloading https://files.pythonhosted.org/packages/b2/5f/23e0023be6bb885d00ffbefad2942bc51a620328ee910f64abe5a8d18dd1/MarkupSafe-1.1.1-cp36-cp36m-manylinux1_x86_64.whl
Building wheels for collected packages: cloudinary
  Building wheel for cloudinary (setup.py): started
  Building wheel for cloudinary (setup.py): finished with status 'done'
  Created wheel for cloudinary: filename=cloudinary-1.15.0-cp36-none-any.whl size=121258 sha256=beea4d0d854ee793adac448cf8a193c0f4f99232d0ef783e43ea219d9e90bead
  Stored in directory: /root/.cache/pip/wheels/21/eb/f0/3810de3ccb6cdebbbe9443202684ff181184eb583c469faa5e
Successfully built cloudinary
Installing collected packages: Werkzeug, itsdangerous, click, MarkupSafe, Jinja2, flask, python-dotenv, gunicorn, numpy, opencv-python, six, mock, urllib3, certifi, cloudinary
Successfully installed Jinja2-2.10.3 MarkupSafe-1.1.1 Werkzeug-0.16.0 certifi-2019.11.28 click-7.0 cloudinary-1.15.0 flask-1.0.2 gunicorn-19.8.1 itsdangerous-1.1.0 mock-3.0.5 numpy-1.16.0 opencv-python-4.0.0.21 python-dotenv-0.8.2 six-1.13.0 urllib3-1.25.7
Removing intermediate container e68bc614dc51
 ---> 902298eb1681
Step 7/9 : ENV PYTHONUNBUFFERED 1
 ---> Running in 9fd40d5c84fb
Removing intermediate container 9fd40d5c84fb
 ---> 90ef9166cb2d
Step 8/9 : COPY . /www/
 ---> feed231658aa
Step 9/9 : CMD gunicorn --bind 0.0.0.0:8080 wsgi
 ---> Running in ce50cafc3492
Removing intermediate container ce50cafc3492
 ---> 8e57ab3c51e5
Successfully built 8e57ab3c51e5
Successfully tagged 127.0.0.1:30400/detector-svc:latest
```

Now push the image to registry

```bash
$ docker push 127.0.0.1:30400/detector-svc:latest
The push refers to repository [127.0.0.1:30400/detector-svc]
Get http://127.0.0.1:30400/v2/: EOF
```

If you got that condition, please add or edit /etc/docker/daemon.json with this following entries, and then restart docker, start container and re-push image

```bash
$ sudo nano /etc/docker/daemon.json
{
  "insecure-registries" : ["127.0.0.1:30400"]
}

$ sudo systemctl restart docker
$ docker start socat-registry
$ docker push 127.0.0.1:30400/detector-svc:latest
The push refers to repository [127.0.0.1:30400/detector-svc]
ce1d1099cb3b: Pushed 
dc29b4a26301: Pushed 
327275792349: Pushed 
50bf2caa2fdc: Pushed 
9964f8231ddb: Pushed 
fc66ec016ace: Pushed 
cf5af8f37e1a: Pushed 
3781431fc41d: Pushed 
99e8bd3efaaf: Pushed 
bee1e39d7c3a: Pushed 
1f59a4b2e206: Pushed 
0ca7f54856c0: Pushed 
ebb9ae013834: Pushed 
latest: digest: sha256:43230a29e679fc9c962056d40cc3d8aa11ad4c920e70a6b9e66293a0ccfdce06 size: 3058
```

![push-1](./registry-push-1.png?raw=true)

#### Viewer Image

```bash
$ docker build -t 127.0.0.1:30400/viewer-svc -f viewer/Dockerfile viewer
Sending build context to Docker daemon  2.485MB
Step 1/3 : FROM php:7.2-apache
7.2-apache: Pulling from library/php
8ec398bc0356: Pull complete 
85cf4fc86478: Pull complete 
970dadf4ccb6: Pull complete 
8c04561117a4: Pull complete 
d6b7434b63a2: Pull complete 
83d8859e9744: Pull complete 
9c3d824d0ad5: Pull complete 
616effb632b1: Pull complete 
20683410f309: Pull complete 
a39e604a3ca4: Pull complete 
0fc05a4fae74: Pull complete 
341e1a9337fe: Pull complete 
26e1dfa87b2c: Pull complete 
82b1c9144c1b: Pull complete 
Digest: sha256:9f7b6d2f54c1209cee00c3eaf93021cb84f03626bad1efcbfc6ff7046954eb0c
Status: Downloaded newer image for php:7.2-apache
 ---> d0e98d20a124
Step 2/3 : EXPOSE 80
 ---> Running in 7614b0ac0703
Removing intermediate container 7614b0ac0703
 ---> 5b986523d241
Step 3/3 : COPY . /var/www/html/
 ---> 058f8ed3adf4
Successfully built 058f8ed3adf4
Successfully tagged 127.0.0.1:30400/viewer-svc:latest
```

Next, push to registry

```bash
$ docker push 127.0.0.1:30400/viewer-svc:latest 
The push refers to repository [127.0.0.1:30400/viewer-svc]
a856cb34cf15: Pushed 
894797c22ed4: Pushed 
42cc48b9b5b5: Pushed 
92d5b838b4f7: Pushed 
49cf737f8b0b: Pushed 
b5b75d49e661: Pushed 
fb5f4891f025: Pushed 
319d9c95668e: Pushed 
e33bff740b75: Pushed 
15533bbdb24d: Pushed 
611630fbca9e: Pushed 
3ff09992e1cd: Pushed 
892bf6a92022: Pushed 
8147eabfca95: Pushed 
556c5fb0d91b: Pushed 
latest: digest: sha256:608ddc0fc23e6b10adec84fd6a4403d82ac71b33325fe23f2fa5bbe7ca9a2655 size: 3452
```

![push-2](./registry-push-2.png?raw=true)

### Generating Secret

As mentioned earlier, Yoloo depends on Cloudinary and Redis. Cloudinary is a cloud-based image/video hosting service and Redis is an in-memory key-value database.

Create an account on [Cloudinary](https://cloudinary.com/) and on [Redis Labs](https://redislabs.com/) (a free managed Redis hosting service).

Cloudinary Console:

![cloudinary-console](./cloudinary-console.jpg?raw=true)

RedisLab Console:

![redislabs-config](./redislabs-config.jpg?raw=true)

Create .env files from the example env files in both services (.env.example) and populate them with your Cloudinary and Redis credentials.

Detector service .env

```bash
$ nano detector/detector-service.env
FLASK_APP=detector.py
FLASK_ENV=production
CLOUDINARY_CLOUD_NAME=somethingawesome
CLOUDINARY_API_KEY=0123456789876543210
CLOUDINARY_API_SECRET=formyappseyesonly
```

Viewer service .env

```bash
$ nano viewer/viewer-service.env
CLOUDINARY_CLOUD_NAME=somethingawesome
CLOUDINARY_API_KEY=0123456789876543210
CLOUDINARY_API_SECRET=formyappseyesonly
DETECTOR_SVC_URL=http://detector-service
REDIS_URL=redis://:password@127.0.0.1:6379
```

Notice the url in DETECTOR_SVC_URL? Kubernetes creates DNS records within the cluster, mapping service names to their IP addresses. So we can use http://detector-service and not have to worry about what IP a service actually uses.

Create k8s Secrets from the .env files in both services.

```bash
$ kubectl create secret generic detector-svc-secrets --from-env-file=detector/detector-service.env
secret/detector-svc-secrets created

$ kubectl create secret generic viewer-svc-secrets --from-env-file=viewer/viewer-service.env
secret/viewer-svc-secrets created
```

You can use the following commands to update the secrets.

```bash
$ kubectl create secret generic detector-svc-secrets --from-env-file=detector/detector-service.env --dry-run -o yaml | kubectl apply -f -
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
secret/detector-svc-secrets configured

$ kubectl create secret generic viewer-svc-secrets --from-env-file=viewer/viewer-service.env --dry-run -o yaml | kubectl apply -f -
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
secret/viewer-svc-secrets configured
```

**NB**: If you want to view the secrets on your k8s cluster (e.g when debugging), you can install the jq utility (https://stedolan.github.io/jq/) and run the following where my-secrets is the name of your k8s secret.

```bash
$ kubectl get secret viewer-svc-secrets -o json | jq '.data | map_values(@base64)'
{
  "CLOUDINARY_API_KEY": "T0RZMU1URTNOVE0yTVRJNU5qTTA=",
  "CLOUDINARY_API_SECRET": "ZDFoNlIyTjFiMlpJVkdKYVkxRTBaemhaVUd4Q1FrWTRNemxy",
  "CLOUDINARY_CLOUD_NAME": "WVhKbWFXRnU=",
  "DETECTOR_SVC_URL": "YUhSMGNEb3ZMMlJsZEdWamRHOXlMWE5sY25acFkyVT0=",
  "REDIS_URL": "Y21Wa2FYTXRNVFF5TVRJdVl6RTFMblZ6TFdWaGMzUXRNUzB5TG1Wak1pNWpiRzkxWkM1eVpXUnBjMnhoWW5NdVkyOXRPakUwTWpFeU9pOHZPbk5TZVd4bmJWWm5NWGd3UTFoS1NHeDZUM0p3ZFVWdlpGTkJPREpaUW5wV1FERXlOeTR3TGpBdU1UbzJNemM1"
}
```

## Deployment Applications

The detector and viewer services contain deployment files, detector-deployment.yaml and viewer-deployment.yaml respectively, which tell k8s what workloads we want to run.

```bash
$ kubectl apply -f detector/detector-deployment.yaml 
deployment.apps/detector-svc-deployment created

$ kubectl apply -f viewer/viewer-deployment.yaml 
deployment.apps/viewer-svc-deployment created
```

Check the deployments

```bash
$ kubectl get deployment
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
detector-svc-deployment   0/3     3            0           2m18s
registry                  1/1     1            1           11h
viewer-svc-deployment     0/2     2            0           30s
```

### Deploying Services

The k8s services (not to be confused with microservices) in detector and viewer, detector-service.yaml and viewer-service.yaml, share traffic among a set of replicas and provide an interface for other applications to access them. The detector service uses the ClusterIP k8s service which exposes the app on a cluster-internal IP. This means detector is only reachable from within the cluster. The viewer service uses the LoadBalancer service which exposes it externally to the outside world.

```bash
$ kubectl apply -f detector/detector-service.yaml 
service/detector-service created

$ kubectl apply -f viewer/viewer-service.yaml 
service/viewer-service created
```

Check the services

```bash
$ kubectl get svc
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
detector-service   ClusterIP      10.96.70.43     <none>        80/TCP           14m
kubernetes         ClusterIP      10.96.0.1       <none>        443/TCP          12h
registry           NodePort       10.96.9.104     <none>        5000:30400/TCP   12h
registry-ui        NodePort       10.96.240.240   <none>        8080:30319/TCP   12h
viewer-service     LoadBalancer   10.96.177.184   <pending>     80:30610/TCP     14m
```

## Result

To check the result, execute

```bash
$ minikube service viewer-service
|-----------|----------------|-------------|----------------------------|
| NAMESPACE |      NAME      | TARGET PORT |            URL             |
|-----------|----------------|-------------|----------------------------|
| default   | viewer-service |             | http://192.168.39.77:30755 |
|-----------|----------------|-------------|----------------------------|
🎉  Opening service default/viewer-service in default browser...
```

This will popup browser, the upload some image file.

The result will be like this:

![result](./result.png?raw=true)

## LICENSE

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

