# Fitness and Meal Planner

This project is for hosting my own recipe manager and fitness journal on my Raspberry Pi to be managed with Dockge.
I plan to use these to help me adjust my diet and track my exercise habits in order to help me shift to a healthier lifestyle.

This project was developed and deployed on a Raspberry Pi, all the commands below should be entered in the terminal window unless otherwise specified.
**Mealie will not run on a 32bit operating system. If your pi is unable to run a 64bit OS you can also follow this guide using a 64bit version of Ubuntu on a physical or virtual machine, the steps should mostly be the same.**

## Setup
Ensure you have wget, SQLite, Docker and Docker Compose installed on your system before proceeding with this project. If you haven't installed them yet, you should be able to get all four with apt:

```
sudo apt install wget

sudo apt install sqlite -y

sudo apt install docker -y

sudo apt install docker-compose -y
```

## Installing Mealie using Docker
Mealie is a self-hosted recipe manager and meal planner with a RestAPI backend and a reactive frontend application built in Vue for a pleasant user experience for the whole family. 
Easily add recipes into your database by providing the url and Mealie will automatically import the relevant data or add a family recipe with the UI editor. 
Mealie also provides an API for interactions from 3rd party application

We will be installing Mealie via docker-compose on the command line. Later we will integrate it with Dockge for seamless management.
First we need to make a directory for mealie and navigate to it:

```
mkdir mealie && cd mealie
```

Create a docker-compose.yml file:

```
nano docker-compose.yml
```

You can paste this inside of docker-compose.yml:

```
version: "3.7"
services:
  mealie:
    image: ghcr.io/mealie-recipes/mealie:v1.3.2 # 
    container_name: mealie
    ports:
        - "9925:9000" # 
    deploy:
      resources:
        limits:
          memory: 1000M # 
    volumes:
      - mealie-data:/app/data/
    environment:
    # Set Backend ENV Variables Here
      - ALLOW_SIGNUP=true
      - PUID=1000
      - PGID=1000
      - TZ=TZ=America/Los_Angeles
      - MAX_WORKERS=1
      - WEB_CONCURRENCY=1
      - BASE_URL=https://mealie.yourdomain.com
    restart: always

volumes:
  mealie-data:
    driver: local
```

This Docker Compose file sets up Mealie in a containerized environment with specific configurations, environment variables, and persistent storage for data.
You can learn more about what this file does in the docker-compose.yml section near the bottom of this readme.

After you've configured the docker-compose.yml files, you can start Mealie by running the following command in the mealie directory.

```
sudo docker-compose up -d
```

The container should start without error and you should be able to access the Mealie frontend at http://localhost:9925 on your pi, or at http://YourPiIPaddress:9925 on a machine in the same network as your pi.

## Basic Mealie Setup

Once you are on the Mealie sign-in page the first thing you should do is change the default credentials 

To do that you will have to log in using the default credentials:

```
Username: changeme@example.com

Password: MyPassword
```

Next click on the "Change Me" profile tab at the top left of the screen. From there Click on "Manage User Profile" which is below "Personal" and "User Settings"

Click on "Change Password", enter the current password (MyPassword) and then enter your new password and confirm it.

Next go back to "Manage User Profile", then change the username, full-name, and email fields to reflect your desired credentials. Then click "Update"

Congratulations! You are now ready to start using Mealie. Please see the reference materials section at the bottom of this readme for a guide on getting started.

## Installing Dockge:

Dockge is a self-hosted Docker stack manager developed by the same person behind the popular software Uptime Kuma.
This software allows you to manage multiple Docker compose files from a single, easy-to-use interface. It is similar to the stack system Portainer implements but cleaner and simpler to use.
You can perform multiple actions within the Dockge interface without touching the command line. For example, you can easily create, edit, start, start, restart, and delete Compose files. You can even update your containers with ease.

First we need to make a directory in /opt for Dockge and a directory for it to manages stacks in, the default directory that Dockge manages stacks in is /opt/stacks so that is what we will be using for this project.

```
mkdir -p /opt/dockge

mkdir -p /opt/stacks 

cd /opt/dockge
```

Once you are in the dockge directory you will need to download the compose.yaml file:

```
sudo curl https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml --output compose.yaml
```

You should see some output that looks like:

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   775  100   775    0     0   1193      0 --:--:-- --:--:-- --:--:--  1194
```

Next you will need to start the server:

```
sudo docker-compose up -d
```

The container should start without error, and you should be able to access the Dockge frontend at http://localhost:5001 on your pi, or at http://YourPiIPaddress:5001 on a machine in the same network as your pi..

Once you've reached the web page you will need to create an account by filling in the fields and clicking "CREATE"

Welcome to Dockge you can use this UI to create new docker-compose stacks and manage existing ones. You can see your stacks on the left of the screen, stacks not being managed by Dockge will be grayed out (Mealie should be visible but grayed out at this time)

Next we are going to integrate Mealie with Dockge.

## Integrating Mealie with Dockge for management.

This is a really simple process that just requires us to move our mealie directory into the /opt/stacks directory we created when we were installing Dockge.

First make sure you create a backup if you have been using Mealie, otherwise you will lose all of your existing data.
Please see https://docs.mealie.io/documentation/getting-started/usage/backups-and-restoring/

Once you have your backup you can bring down Mealie by navigating to the mealie directory and using:

```
sudo docker-compose down -v
```

Once mealie is down you will need to move it to the /opt/stacks directory. To do this, navigate back out to your home directory, or whatever directory your mealie project folder is in and move it to the new directory.

```
cd ..

mv mealie /opt/stacks
```

Now return to your Dockge frontend at http://localhost:5001 on your pi, or at http://YourPiIPaddress:5001 on a machine in the same network as your pi..
On the left you should see that mealie is no longer grayed out. Click on mealie and then click "Start" at the top of the page. 
Once Mealie has launched simply sign in using the default credentials (Username: changeme@example.com, Password: MyPassword) and restore your backup!

You can now manage your Mealie stack from Dockge!

Next we will use Dockge to deploy a docker-compose stack named ExerciseDiary.

## Installing ExerciseDiary with Dockge:

ExerciseDiary is a self-hosted solution for keeping track of your workouts and weight using an easy-to-use web interface.
This exercise diary gives you a neat way of visualizing your workouts using a GitHub-style visualization. This visualization is a heap map that gives you an easy way of seeing days you worked out harder than others.
In addition to the workout diary, the ExerciseDiary software also tracks your weight, giving you a simple line graph to track your progress.

First, return to your Dockge frontend and click "+Compose" at the top left of the page. 

For Stack Name I am going to name this stack "exdiary", 

We'll leave Dockge Agent at its default setting "(online) Current"

There will be an nginx container under the "Containers" section by default. Anytime you compose a stack with Dockge it will already an nginx container with default configuration ready to go.
For the purpose of this project we are just going to leave the nginx container with it's default settings in the stack, you can remove it you want by clicking "Delete" under "nginx"

Then give your container a name by filling out the field under the "Containers" section and then click the "Add Container" button.

We're going to fill in a few fields before this stack deploys. We will be using the docker-compose.yml file on the ExerciseDiary github page (https://github.com/aceberg/ExerciseDiary/blob/main/docker-compose.yml)

So our image will be: **aceberg/exercisediary**

Click on "Add Port" and enter: **8851:8851 into the generated field.**

Next click "Add Volume" and enter: **~/.dockerdata/ExerciseDiary:/data/ExerciseDiary**

We'll be leaving the Restart Policy as its default: **Unless Stopped**

Next click "Add Environment Variable" five times. Then one line per field, add these lines to the generated fields:


```
"TZ: America/Los_Angeles"

'HOST: "0.0.0.0"'

'PORT: "8851"'

'THEME: "grass" '

'COLOR: "light" '
```

We will not be configuring any Networks, Dependencies, or URLs for this project so we will leave all of those blank.

You may have noticed the docker-compose.yml file generating in the top right of your screen while you enter this information, it should now look like this:

```
version: "3.8"
services:
  nginx:
    image: nginx:latest
    restart: unless-stopped
    ports:
      - 8080:80
  exercisediary:
    restart: unless-stopped
    image: aceberg/exercisediary
    ports:
      - 8851:8851
    volumes:
      - ~/.dockerdata/ExerciseDiary:/data/ExerciseDiary
    environment:
      - "TZ: America/Los_Angeles"
      - 'HOST: "0.0.0.0"'
      - 'PORT: "8851"'
      - 'THEME: "grass" '
      - 'COLOR: "light" '
    networks: []
networks: {}
```

Once you have verified that everything is set up correctly you can scroll back up to the top of the screen and click "Deploy"

The container should start without error and you should be able to access the ExerciseDiary frontend at http://localhost:8851 on your pi, or at http://YourPiIPaddress:8851 on a machine in the same network as your pi.

You should also be redirected to a dashboard that shows your containers with their status and respective ports, and a terminal window that you can use to monitor the stack.

From this page you can easily manage your docker-compose stack as a whole or each container within individually, there are also some really neat tools.

At the top of the page are options to Edit, Restart, Update, Stop, or Delete your stack. There are also links next to each container that say ">_Bash" which will take you to a bash shell in the container, and a copy of the docker-compose.yml on the right of the page.

Congratulations on deploying your first stack using the Dockge interface. I hope you found this project informative!

Below is a brief overview of the docker-compose.yml files we wrote in this project, and some reference material for getting started with your newly created containers.

# Compose files:
## Mealie - docker-compose.yml:

This gives a brief overview of the docker-compose.yml file for mealie.

```
version: "3.7"
```

This line specifies the version of the Docker Compose file format being used. In this case, it's version 3.7.

```
services:
  mealie:
```

This section defines a service named "mealie", which is the containerized instance of the Mealie application.

```
    image: ghcr.io/mealie-recipes/mealie:v1.3.2
```

This line specifies the Docker image to be used for the "mealie" service. It pulls the image from the GitHub Container Registry (ghcr.io) and uses the version tagged as "v1.3.2".

```
    container_name: mealie
This sets the custom name for the container to "mealie". Containers created from this service will be identified with this name.
```
```
    ports:
        - "9925:9000"
```

This line maps port 9925 on the host machine to port 9000 inside the container. It allows external access to the Mealie application through port 9925.

```
    deploy:
      resources:
        limits:
          memory: 1000M
```

This section is used for Docker Swarm deployment configurations. It sets a memory limit of 1000 megabytes for the "mealie" service.

```
    volumes:
      - mealie-data:/app/data/
```
This line creates a Docker volume named "mealie-data" and mounts it to the "/app/data/" directory inside the container. This volume is used for persistent storage of Mealie data.

```
    environment:
      - ALLOW_SIGNUP=true
      - PUID=1000
      - PGID=1000
      - TZ=America/Anchorage
      - MAX_WORKERS=1
      - WEB_CONCURRENCY=1
      - BASE_URL=https://mealie.yourdomain.com
```
These environment variables are set for the Mealie container:

ALLOW_SIGNUP=true: Allows user signup in Mealie.

PUID=1000: Specifies the user ID for file permissions inside the container.

PGID=1000: Specifies the group ID for file permissions inside the container.

TZ=America/Anchorage: Sets the timezone to America/Anchorage.

MAX_WORKERS=1: Sets the maximum number of worker processes to 1.

WEB_CONCURRENCY=1: Sets the web concurrency to 1.

BASE_URL=https://mealie.yourdomain.com: Specifies the base URL for Mealie. Replace "mealie.yourdomain.com" with the actual domain name.

```
    restart: always
```

This line ensures that the "mealie" container restarts automatically if it exits unexpectedly.

```
volumes:
  mealie-data:
    driver: local
```
This section defines a Docker volume named "mealie-data" with a local driver. It's used to persist Mealie data even if the container is stopped or deleted.

## Dockge - compose.yaml

This gives a brief overview of the docker-compose.yml file we downloaded using wget when installing Dockge.

```
version: "3.8"
```

This line specifies the version of the Docker Compose file format being used. In this case, it's version 3.8.

```
services:
  dockge:
```

This section defines a service named "dockge", which is the containerized instance of the Dockge application.

```
    image: louislam/dockge:1
```

This line specifies the Docker image to be used for the "dockge" service. It pulls the image from the Docker Hub repository louislam/dockge with version tag 1.

```
    restart: unless-stopped
```

This line ensures that the "dockge" container restarts automatically if it exits unexpectedly, unless explicitly stopped.

```
    ports:
      - 5001:5001
```

This line maps port 5001 on the host machine to port 5001 inside the container. It allows external access to the Dockge application through port 5001.

```
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

This line mounts the Docker socket file from the host machine to the Docker socket file inside the container. It allows Dockge to communicate with the Docker daemon running on the host.

```
      - ./data:/app/data
```

This line mounts the local directory ./data to the /app/data directory inside the container. It is used for persisting Dockge data.

```
      - /opt/stacks:/opt/stacks
```

This line mounts the directory /opt/stacks from the host machine to the directory /opt/stacks inside the container. It specifies the location of Dockge's stacks directory.

```
    environment:
      - DOCKGE_STACKS_DIR=/opt/stacks

```
This environment variable tells Dockge the location of the stacks directory within the container.

For advanced configuration options and further customization, you can explore Dockge's documentation or adjust the docker-compose.yml file as needed.


## ExerciseDiary - docker-compose.yml

This gives a brief overview of the docker-compose.yml file we made using Dockge.

```
version: "3.8"
```

This line specifies the version of the Docker Compose file format being used. In this case, it's version 3.8.

```
services:
  nginx:
    image: nginx:latest
    restart: unless-stopped
    ports:
      - 8080:80
```

This subsection defines the NGINX service.

image: nginx:latest: Specifies the Docker image to use for the NGINX service. In this case, it uses the latest version of the NGINX image from the Docker Hub.

restart: unless-stopped: Configures the NGINX container to restart automatically unless it is explicitly stopped.

ports: - 8080:80: Maps port 8080 on the host machine to port 80 inside the NGINX container, allowing traffic to be routed to NGINX.

```
    exercisediary:
    restart: unless-stopped
    image: aceberg/exercisediary
    ports:
      - 8851:8851
    volumes:
      - ~/.dockerdata/ExerciseDiary:/data/ExerciseDiary
```

This subsection defines the ExerciseDiary service.

restart: unless-stopped: Configures the ExerciseDiary container to restart automatically unless it is explicitly stopped.

image: aceberg/exercisediary: Specifies the Docker image to use for the ExerciseDiary service. It pulls the image named aceberg/exercisediary from a Docker registry.

ports: - 8851:8851: Maps port 8851 on the host machine to port 8851 inside the ExerciseDiary container, allowing traffic to be routed to ExerciseDiary.

volumes: - ~/.dockerdata/ExerciseDiary:/data/ExerciseDiary: Mounts the directory ~/.dockerdata/ExerciseDiary on the host machine to the directory /data/ExerciseDiary inside the ExerciseDiary container, allowing persistent storage of ExerciseDiary data.


```
    environment:
      - "TZ: America/Los_Angeles"
      - 'HOST: "0.0.0.0"'
      - 'PORT: "8851"'
      - 'THEME: "grass" '
      - 'COLOR: "light" '
```

"TZ: America/Los_Angeles": this environment variable sets the timezone of the container to "America/Los_Angeles". It determines how dates and times are displayed within the containerized application.

'HOST: "0.0.0.0"': this environment variable sets the host IP address or hostname that the ExerciseDiary application will bind to. Setting it to "0.0.0.0" means that the application will listen on all network interfaces, allowing it to accept connections from any available network interface on the container host.

'PORT: "8851"': this environment variable specifies the port number on which the ExerciseDiary application will listen for incoming connections.

'THEME: "grass"': this environment variable defines the theme of the ExerciseDiary application. It sets the theme to "grass"

'COLOR: "light"': this environment variable specifies the color scheme of the ExerciseDiary application. It sets the color to "light"


# Reference material:

Here are links to the guides that I used to help me make this project.

## Installation guides

https://docs.mealie.io/documentation/getting-started/installation/installation-checklist/

https://pimylifeup.com/raspberry-pi-dockge/

https://pimylifeup.com/raspberry-pi-exercisediary/

## Getting started with these services

https://docs.mealie.io/documentation/getting-started/introduction/

https://github.com/louislam/dockge

https://quibtech.com/p/dockge-a-new-stack-manager/

https://github.com/aceberg/ExerciseDiary

## More on docker and docker-compose

https://docs.docker.com/compose/
