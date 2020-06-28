Every now and then I come across the requirement to build Docker images inside a Docker container. More often than not, this happens when I need to build Docker images as part of a Continuous Integration pipeline running Jenkins - where the Jenkins master (or agent) is running inside a Docker container.

Docker doesn’t recommend running the Docker daemon inside a container (except for very few use cases like developing Docker itself), and the solutions to make this happen are generally hacky and/or unreliable.

Fear not though, there is an easy workaround: mount the host machine’s Docker socket in the container. This will allow your container to use the host machine’s Docker daemon to run containers and build images.

Your container still needs compatible Docker client binaries in it, but I have found this to be acceptable for all my use cases.

The TL;DR version using my prebuilt image is:

docker run \
  -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name jenkins \
  getintodevops/jenkins-withdocker:lts
The guide below is for Jenkins, but you can apply the same logic to any other build server.

Building Docker containers with Jenkins inside a container
First, we’ll run Jenkins as a container using the official Jenkins image:

docker run -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name jenkins \
  jenkins/jenkins:lts
Note that the key here is mounting /var/run/docker.sock from the host machine to the same location inside the container.

Then, we’ll need to install the Docker binaries inside the container. Spawn an interactive shell inside the running Jenkins container:

docker exec -it -u root jenkins bash
Because the official Jenkins image is based on Debian 9, we can use apt to install the Docker binaries as instructed in the Docker installation guide. This is a single snippet to install some prerequisites, configure the official Docker apt repositories and install the latest Docker CE binaries:

apt-get update && \
apt-get -y install apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common && \
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable" && \
apt-get update && \
apt-get -y install docker-ce
Releaseworks Academy has a free online training course on Docker & Jenkins best practices: https://www.releaseworksacademy.com/courses/best-practices-docker-jenkins

Caveat: The Docker daemon running on your host machine must be compatible with the version of client binaries you are installing. To verify the version, run docker version on your host machine.

Ta-da! Your Jenkins container should now have a functioning Docker installation. Verify by running:

docker ps
The output should be the same as when running the command on the host machine (you should at least expect to see the Jenkins container running!).

Next, you’ll need to finish the installation of Jenkins as usual. Open the Jenkins installer by navigating to http://localhost:8080.

You will need the initial admin password, which can be obtained by running:

docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
Next select “Install recommended plugins”, and wait for Jenkins to install everything. The Docker Pipeline plugin is installed by default, which means you are ready to go!

I have pre-built a Jenkins 2.73.1 image with Docker 17.09.0-ce binaries. It’s available on Docker Hub as getintodevops/jenkins-withdocker: https://hub.docker.com/repository/docker/getintodevops/jenkins-withdocker.