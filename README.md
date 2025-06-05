# Run an application in docker container by Jenkins
## 📌 Project Overview
The class practice includes:
- Launch the docker composed based lab from [jenkins-shallow-dive](https://gitlab.com/vaiolabs-io/jenkins-shallow-dive)
- Define agent as docker in [Jenkinsfile](./Jenkinsfile)
- Run and test a simple [application](https://gitlab.com/silent-mobius/my_awesome_app) (Flask+python) in docker Jenkins node

## Prerequisite
- Linux node host
- Installed packages: bash, git and docker
- Write access to [GitLab](https://gitlab.com)
- Account in [DockerHub](https://hub.docker.com)

## 👣 Steps
The steps below describe whole the process I pass in order to arrive the final target when the application runs in the docker node and provides the correct response.
### 1. Prepare the application
- Fork the [application](https://gitlab.com/silent-mobius/my_awesome_app) repository to my GitLab [account](https://gitlab.com/baruch.gudesblat)  
- Verify the `debug` mode in application is disabled. File app.py - `debug=False`

### 2. Clone the Jenkins course repository to local Host  
`git clone https://gitlab.com/vaiolabs-io/jenkins-shallow-dive`

### 3 Prepare custom image with git and sudo for Jenkins node
_Build your image with the steps below or, alternively, You are welcome to use my image_ `baruchgu/jenkins-agent-with-sudo`  
<details><summary>Build your image</summary>

  3.1 Add new file Dockerfile.deb.agent-with-sudo  
```sh
cat < EOI > Dockerfile.deb.agent-with-sudo  
FROM jenkins/inbound-agent  
USER root  
RUN apt-get update && apt-get install -y sudo git  
RUN echo "jenkins ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers  
USER jenkins  
EOI
```  
  3.2 Build the image (modify mycompany to your account in [GitHub](https://hub.docker.com/))  
`docker build -t mycompany/jenkins-agent-with-sudo -f Dockerfile.deb.agent-with-sudo .`  
  3.3 Upload the image to GitHub repository
`docker login -u mycompany`  
`docker push mycompany/jenkins-agent-with-sudo`    
</details>

### 4. Launch Jenkins docker containers   
  4.1 `cd jenkins-shallow-dive/99_misc/setup/docker`  
  4.2 Modify Dockerfile.deb.worker  
    - remove lines with "&& TEMP_DEB" and "&& dpkg -i ...", lines 5,6  
    - add `curl` to `apt-get install` arguments, line 3  
  4.3 Modify Dockerfile.main  
    - remove lines with "&& TEMP_DEB" and "&& dpkg -i ...", lines 5,6  
    - remove line `RUN jenkins-plugin-cli --plugins`  
    - add this at end `RUN mkdir /etc/docker && echo '{ "dns": ["8.8.8.8", "8.8.4.4"] }' > /etc/docker/daemon.json`   
  4.4 Open the permission on the Host linux  
  `sudo chmod a+r /var/run/docker.sock`  
  4.5 Open DNS in the Host linux
  ```sh
  mkdir /etc/docker
  echo '{ "dns": ["8.8.8.8", "1.1.1.1"] }' > /etc/docker/daemon.json
  ```
  4.6 Launch the docker compose  
  `docker compose up -d`

### 5. Prepare Jenkinsfile in this repository
  5.1 Define agent with 
  ```
   	agent { dockerContainer {  
			image 'baruchgu/jenkins-agent-with-sudo'  
		} }  
  ```
  5.2 Add `binutils` and `python3-dev` to `apt-get install` line in `Pre-Build` stage  
  5.3 Add parameter `sleep_time`   
```
parameters {  
	string(name: 'sleep_time', defaultValue: "4", description: 'time to sleep after build stage')  
}   
```  
  5.4 Add this at top of the `Test` stage script, use triple """ quotes surround  
  ```sh
  set -e
  python3 app.py &
  sleep ${params.sleep_time}  
  ```
  5.5 Modify the application port at `curl localhost:8000`  
  5.6 Commit the changes to GitLab

### 6. Launch and setup Jenkins server
6.1 Launch the browser `firefox http://localhost:80`  
6.2 Install Plugins at `http://localhost/manage/pluginManager/available`
  - GitLab Plugin  
  - Docker plugin  
  - Docker Pipeline  

6.3 Open New cloud at `http://localhost/manage/cloud/`  
  - Name - as you wish  , ex: docker_agent
  - `Docker Host URI` set `unix:///var/run/docker.sock`  

6.4 Open New Item `my_awesome_app` as pipeline type with configurations:
  - `Definition` choose `Pipeline script from SCM`
  - `SCM` choose `GIT`
  - `Repository URL` set your (or my) GitLab repo. Ex: `https://gitlab.com/baruch.gudesblat/my_awesome_app.git`
  - `Credentials` -none-
  - `Branch` set `*/main`
  - `Script Path` as a default `Jenkinsfile`  

Note: the python application app.py is installed in the pipeline workspace as a side effect when Jenkins fetches whole the repository as is defined in `Repository URL`

### 7. Run the pipeline Build
- Browse to `http://localhost/job/my_awesome_app/`
- Click `Build with Parameters`
- Expect the green success mark

## 👥 Contributors
[Baruch](https://github.com/baruchgu) - Owner

## 🌐 Links
* [Jenkins Shallow Dive course](https://gitlab.com/vaiolabs-io/jenkins-shallow-dive)
* [Awesome application](https://gitlab.com/silent-mobius/my_awesome_app)
* [Docker Hub](https://www.hub.docker.com)
* [GitLab](https://gitlab.com)

