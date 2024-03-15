Docker is a tool for creating and managing containers

Container : A package of code and dependeincies and execution behaviour.

Installing docker in linux :

    sudo yum update -y
    sudo yum -y install docker
     
    sudo service docker start
     
    sudo usermod -a -G docker ec2-user

To start service :
  sudo systemctl enable docker


Basic Commands : 
- docker version
- docker info
- docker <<Command>> <<sub-command>> (options)


Image is the application we want to run while a container is an instance of that image running as process.

# Container commands
> docker container run --publish 80:80 nginx 
> docker container run -p 80:80 nginx
- '--publish 80:80 or -p 80:80' to export container internal port to host port
> docker container run -p 80:80 --detach --name webhost nginx
- '--detach  or -d' to detach container from current prompt
- docker run -d ubuntu:18.04 tail -f /dev/null : Run in detached mode and discard all output. <a href='https://www.geeksforgeeks.org/what-is-dev-null-in-linux/'>dev/null</a>
- '--name webhost' to give container a unique name, if not specified docker will pick a random name and assign to container

> To check logs
  docker container logs webhost  

> to list out processes running inside container  
  docker container top webhost

> To Stop container  
  docker container stop <<Few_character_of_Container_id>>

> To start container  
  docker container start <<Container_name>>
  
> To list all containers   
  docker container ls 
  'ls -a' to list all stopped or running containers

> To remove container  
  docker container rm <<container_id>>
  Attribute 'rm' is to remove container is it is stopped
  '-f' will forcefully remove container
  
> Running container in interactive mode     
  docker run -it ubuntu  
  Above command will start a container and give us a Shell where we can execute command  
  '–rm' argument when we start a container in interactive mode. It’ll make sure to remove the container when we exit: docker run -it --rm ubuntu:18.04  
  
> To connect existing running container :
  docker exec -it <<Container_id>> sh   
  'docker exec' tells docker that we want to execute command into running container  
  '-it' argument means  that it will be executed in interactive mode '-it' keeps the STDIN open  
      -i --interactive to keep STDIN open even if not attached.  
      -t --tty : Allocates a pseudo TTY (Creates a terminal)
  'sh' is the command we want to execute  




> docker container create (or shorthand: docker create) command creates a new container from the specified image, without starting it.
  

<a href="https://docs.docker.com/reference/cli/docker/container/">Container commands</a>