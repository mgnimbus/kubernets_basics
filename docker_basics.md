# Docker
Meet Docker!

* First release was march 20,2103
* written in Golang that utilizes the host os
* dockers uses linux kernal, built on LXC
* uses a filesystem called **overlay2**
  * File needed for containere are s* Tored in plainsight
* Docker containers are essentially system processisolated by **cgroups**
  
! Insert images here

## Docker Commands
* Tree
  ```
  tree -L 1 /dir  # To view the level 1 directories

  tree -L 2       # To view the level 2 directories
  ```
* PS
  ```
  ps -aux       # To view all the process running in liux

  docker ps    # just show all the docker container that are runnnig
  ```  

* To remove all images stored locally
  ```docker
  docker rmi -f $(docker images -a -q) # To delete all the images in the docker
  ```

* To search for available python images in docker hub

  ```
  docker search python
  docker search --filter is-official=true
  ```

* To limit the search for the image type --filter/-f(shorthand) **flag**
  ```
  docker search --filter is-official=true
  ```

* To get the help for a docker <command> --help/-h
  ```
  docker search --help
  ```

  ```
  response:
    -f, --filter filter   Filter output based on conditions provided
        --format string   Pretty-print search using a Go template
        --limit int       Max number of search results
        --no-trunc        Don't truncate output
  ```
* To pull the docker image locally from docker hub

  ```
  docker pull python # by def it pulls the latest images
  ```
  * To list the all the docker **images**  available locally
  ```
  docker images 
  ```

* To pull the docker **Image ID** 
  ```
  docker images -q
  ```
* To see all the layers in the image docker file
  ```
  docker history <imageID>

  docker history $(docker images -q | head -n 1) # To fetch the layers from top of images 

  $(docker images -q | head -n 1) is the sub-command to get image id
  ```
  
* To extact the req fields and export  
  ```
  docker history --format "{{.ID}}: {{.CreatedBy}}: {{.Size}}" $(docker images -q | head -n 1) > python_image_layers
  ```
* Lets see the diffences b/w pulls 
  ```
  docker pull python:3.9
  *layers*
  de4cac68b616: Already exists 
  d31b0195ec5f: Already exists 
  9b1fd34c30b7: Already exists 
  c485c4ba3831: Already exists 
  9c94b131279a: Already exists 
  252df837ef06: Pull complete 
  836bdd8516b8: Pull complete 
  e1bb02cffbaf: Pull complete
  ```
  * If you see the docker engine is using the already existing layers in locall form latest version and building only that are need for 3.9
  * see the diff in the exported files

* Now, Docker pull just pull the images from docker hub and store them locally there are like CD's just there for you so you can mount them and run
* To run the image you need a container like a cd need a reader to run

* To create a container
  ```
  docker create <image>
  ``` 
  * The above command just creates a new container from the specified image, without starting it.

* To see all docker containers running
  ```
  docker ps 
  ``` 
* To see all the docker container that are present need to use flag --a
  ```
  docker ps --all --size
  ```
* To start the docker container
  ```
  docker start <containerID>/<name>
  ```
  * If you docker ps you see that the container is not running 
  * Its because the docker container is a emfermal as you are nor ruuning and program if exits after seeing there nothing to run


* To rename a docker container
  ```
  docker rename <old_name> <desired_name>
  ```

 * If you use a docker run command it docker create + docker start 
 * docker run create a new contrainer for the image and runs its
 * Its not suitable for production so its better to create a contianer and start when you need
    ```
    docker run <image>
    ```

* To run a command in python conatiner 

  ```
  docker run <image> python -c 'print("Hola, Docker!")'
  python -c                                             # runs python command in cli
  ```
  * To see the difference

  ```
  python3 --version
  docker run python python3 --version
  ```

  * Everytime you can run `docker run <image>` a container will be created
  <br>
<br>

* `docker run --interactive python`   - keeps STDIN open even its not attached
* `docker run --tty python`           - allocate a psuedo a terminal
* `docker run -it python`             - shorthand

* It open a interactive terminal to the container to use
  ```
  docker run python -it

  >>> print('hello from docker!')
  >>> exit() or `ctrl + D` an escape sequence
  ```

 * `docker run --name mypy -it python` - to create a container with name mypy
 * `docker run --name mypy -it python` - if you run the same command again it will fail as there is already a container with the same name
 * `docker run --rm --name mypy -it python` - If you want to delete the cont after the invoke 
 * `docker run --rm --name mypy -itd python` - to keep the caontainer running event after exit
 * you can see `docker ps` that the conatainer is still running 
  
 * `docker run --rm --name mypy2 -itd python` -To create a mypy2 container in detached mode (will be running)
 * `docker exec -it mypy2 python` - To go to the container
 
 * `docker run --name python_shell -it python`
 * `docker exec -it python_shell /bin/sh` # To go the terminal directly

* `docker start <image>` + `docker attach <image>` will take you to the shell directly but will shutdown while exiting  

* To see the meta data of the docker
  ```
  docker inspect <image>
  ```

* To remove a container 
  ```
  docker remove <image>
  ``` 
! Note: A container must be in stop status to remove a container 

* `docker ps -a -l` - Gives the latest container deatils  
* `docker ps -a -l -q` - Gives the container ID of the latest container
* `docekr rm $(docker ps -a -l -q)` - Removes the latest container 

* `docker ps -a -l -f status=exited` - To get details of the latest exited status container
* `docker rm $(docker ps -a -l -q -f status=exited)` - Removes the latest exited status container

* `docker ps -a -f name=python` # To get the details of the container with name filter
* `docker rm $(docker ps -a -q -f name=python)`

* `docker ps -a -f ancestor=python` # To get the details of all the container with base image as  python

| commands                                                                              | description |
| ------------------------------------------------------------------------------------- | ----------- |
| `docker ps -a -l`                                                                     | hello       |
| `docker ps -a -l -q`                                                                  |             |
| `docker ps -a -l --format {{.Names}}`                                                 |             |
| `docker ps -a --filter before=$(docker ps -a -l --format {{.Names}})`                 |             |
| `docker rm $(docker ps -a -q --filter before=$(docker ps -a -l --format {{.Names}}))` |             |


| commands                            | description                         |
| ----------------------------------- | ----------------------------------- |
| `docker rmi -f $(docker images -q)` | To remove all the images forcefully |


| commands                      | description |
| ----------------------------- | ----------- |
| `docker pull <image_name:tag` |             |
| `docker pull python`          |             |
| `docker `                     |             |
| `docker `                     |             |
| `docker `                     |             |
| `docker `                     |             |
| `docker `                     |             |
| `docker `                     |             |
| `docker `                     |             |
| `docker `                     |             |

## Lets run an application in the docker using pyhton

* create a directory /infrastructure/dev1/app where the app code is stored
* `docker run -it --name mypthon python` 
* `docker start $(docker -alq)`
* 
* `docker exec -it $(docker -al --format {{.Names}}) /bin/sh`
* `docker exec -itd $(docker -al --format {{.Names}}) /bin/sh`  
* 
* `docker run -it --name mypthon2 python /bin/sh`
* `docker run -itd --name mypthon2 python /bin/sh`  

* Now we need bind mount our local app directory to docker container

`docker run -it -v $PWD/app:/app python /bin/sh`
`docker run -it -v $PWD/app:/app --name mypy python /bin/sh` 

* Now you app dir is binded to the docker container
* run `pip install -r requirements.txt` in app dir
* run `python app.py`

* we can automate the by creating a run.sh file to run the both commands while booting
* `docker run -it -v $PWD/app:/app --name mypy2 python /bin/sh run.sh` 
* Error: /bin/sh: 0: cannot open run.sh: No such file

* `docker run -it -v $PWD/app:/app -w /app --name mypy2 python /bin/sh run.sh` - To point to the working to run the script