# student-list project

Please find the specifications by clicking [here](https://github.com/diranetafen/student-list.git "here")

!["Crédit image : eazytraining.fr"](https://eazytraining.fr/wp-content/uploads/2020/04/pozos-logo.png) ![project](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)

------------

Firstname : Abdel-had

Surname : HANAMI

For Eazytraining's 12th DevOps Bootcamp

Period : march-april-may

Sunday the 12th, march 2023



## Application

I had to deploy an application named "*student_list*", which is very basic and enables POZOS to show the list of some students with their age.

student_list application has two modules:

- The first module is a REST API (with basic authentication needed) who send the desire list of the student based on JSON file
- The second module is a web app written in HTML + PHP who enable end-user to get a list of students


## The need

My work was to :
1) build one container for each module
2) make them interact with each other
3) provide a private registry


## My plan

First, let me introduce you the six ***files*** of this project and their role 

Then, I'll show you how I ***built*** and tested the architecture to justify my choices

Third and last part will be about to provide the ***deployment*** process I suggest for this application.


### The files' role

In my delivery you can find three main files : a ***Dockerfile***, a ***docker-compose.yml*** and a ***docker-compose.registry.yml***

- docker-compose.yml: to launch the application (API and web app)
- docker-compose.registry.yml: to launch the local registry and its frontend
- simple_api/student_age.py: contains the source code of the API in python
- simple_api/Dockerfile: to build the API image with the source code in it
- simple_api/student_age.json: contains student name with age on JSON format
- index.php: PHP  page where end-user will be connected to interact with the service to list students with their age.


## Build and test

Considering you just have cloned this repository, you have to follow those steps to get the 'student_list' application ready :

1) Change directory and build the api container image :

```bash
cd ./mini-projet-docker/simple_api
docker build . -t api.student_list.img
docker images
```
> ![1-docker images](https://user-images.githubusercontent.com/101605739/224588377-b8afa11f-33b6-41ed-9f58-6e23d2054c83.jpg)

2) Create a bridge-type network for the two containers to be able to contact each other by their names thanks to dns functions :

```bash
docker network create student_list.network --driver=bridge
docker network ls
```
> ![2-docker network ls](https://user-images.githubusercontent.com/101605739/224588523-a842cd26-c5d5-4338-8547-2e31578655c9.jpg)


3) Move back to the root dir of the project and run the backend api container with those arguments :

```bash
cd ..
docker run --rm -d --name=api.student_list --network=student_list.network -v ./simple_api/:/data/ api.student_list.img
docker ps
```
> ![3-docker ps](https://user-images.githubusercontent.com/101605739/224589378-abcc3f7d-d5c6-4a81-ba28-767cb6cd7b7c.jpg)

As you can see, the api backend container is listening to the 5000 port.
This internal port can be reached by another container from the same network so I chose not to expose it.

I also had to mount the `./simple_api/` local directory in the `/data/` internal container directory so the api can use the `student_age.json` list 


> ![4-./simple_api/:/data/](https://user-images.githubusercontent.com/101605739/224589839-7a5d47e6-fdff-40e4-a803-99ebc9d70b03.png)


4) Update the `index.php` file :

You need to update the following line before running the website container to make ***api_ip_or_name*** and ***port*** fit your deployment
   ` $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';`

Thanks to our bridge-type network's dns functions, we can easyly use the api container name with the port we saw just before to adapt our website

```bash
sed -i s\<api_ip_or_name:port>\api.student_list:5000\g ./website/index.php
```
> ![5-api.student_list:5000](https://user-images.githubusercontent.com/101605739/224590958-49c2ce64-c9a0-4655-93da-552f27f78b2f.png)


5) Run the frontend webapp container :

Username and password are provided in the source code `.simple_api/student_age.py`

> ![6-id/passwd](https://user-images.githubusercontent.com/101605739/224590363-0fdd56ae-9fb9-45e7-8912-64a6789faa9e.png)

```bash
docker run --rm -d --name=webapp.student_list -p 80:80 --network=student_list.network -v ./website/:/var/www/html -e USERNAME=toto -e PASSWORD=python php:apache
docker ps
```
> ![7-docker ps](https://user-images.githubusercontent.com/101605739/224591443-344fd2cd-ddbc-4780-bbc5-7cc0bdac156f.jpg)


6) Test the api through the frontend :

6a) Using command line :

The next command will ask the frontend container to request the backend api and show you the output back.
The goal is to test both if the api works and if frontend can get the student list from it.

```bash
docker exec webapp.student_list curl -u toto:python -X GET http://api.student_list:5000/pozos/api/v1.0/get_student_ages
```
> ![8-docker exec](https://user-images.githubusercontent.com/101605739/224593842-23c7f3a5-e5bc-4840-a6af-2eda0f622710.png)


6b) Using a web browser `IP:80` :

- If you're running the app into a remote server or a virtual machine (e.g provisionned by eazytraining's vagrant file), please find your ip address typing `hostname -I`
> ![9-hostname -I](https://user-images.githubusercontent.com/101605739/224594393-841a5544-7914-4b4f-91fd-90ce23200156.jpg)

- If you are working on PlayWithDocker, just `open the 80 port` on the gui
- If not, type `localhost:80`

Click the button

> ![10-check webpage](https://user-images.githubusercontent.com/101605739/224594989-0cb5bcb7-d033-4969-a12e-0b2aa9953a97.jpg)


7) Clean the workspace :

Thanks to the `--rm` argument we used while starting our containers, they will be removed as they stop.
Remove the network previously created.


```bash
docker stop api.student_list
docker stop webapp.student_list
docker network rm student_list.network
docker network ls
docker ps
```
> ![11-clean-up](https://user-images.githubusercontent.com/101605739/224595124-3ea15f42-e6d5-462a-92a0-52af7c73c17a.jpg)




## Deployment

As the tests passed we can now 'composerize' our infrastructure by putting the `docker run` parameters into a `docker-compose.yml` code.

1) Run the application (api + webapp) :

As we've already created the application image, now you just have to run :

```bash
docker-compose up -d
```

Docker-compose permits to chose which container must start first.
The api container will be first as I specified that the webapp `depends_on:` it.
> ![12-depends on](https://user-images.githubusercontent.com/101605739/224595564-e010cc3f-700b-4b3e-9251-904dafbe4067.png)

And the application works :
> ![13-check app](https://user-images.githubusercontent.com/101605739/224596825-61b2e2af-ebea-42c0-be8b-20b4743f1ece.jpg)


2) Create a registry and its frontend

I used `registry:2` image for the registry, and `joxit/docker-registry-ui:static` for its frontend gui and passed some environment variables :

> ![14-gui registry env var](https://user-images.githubusercontent.com/101605739/224596117-76cda01c-f2f6-4a18-862f-95d56449f98a.png)


E.g we'll be able to delete images from the registry via the gui.

```bash
docker-compose -f docker-compose.registry.yml up -d
```
> ![15-check gui reg](https://user-images.githubusercontent.com/101605739/224596652-70c2f273-5ec9-406f-b8b5-ea1398b88998.jpg)


3) Push an image on the registry and test the gui

You have to rename it before (`:latest` is optional) :

```bash
docker image tag api.student_list.img:latest pozos-registry:5000/pozos/api.student_list.img:latest
docker images
docker image push pozos-registry:5000/pozos/api.student_list.img:latest
```

> ![16-push image to registry](https://user-images.githubusercontent.com/101605739/224596478-a544269c-5cee-4e90-ace0-fa31a005a429.jpg)

> ![17-full reg](https://user-images.githubusercontent.com/101605739/224597957-e4da3cb4-9c95-4f16-96c7-ffceed3fb558.jpg)

> ![18-full reg details](https://user-images.githubusercontent.com/101605739/224598011-4b2efc75-dfb6-407e-b6cb-0d03a35c09ed.jpg)


------------


# This concludes my Docker mini-project run report.
![octocat](https://myoctocat.com/assets/images/base-octocat.svg)
