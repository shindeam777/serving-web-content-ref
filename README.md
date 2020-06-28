docker build -t jensonjoseph/spring-boot-docker .
docker run -p 9090:8080 jensonjoseph/spring-boot-docker

https://github.com/jenkinsci/docker
docker run -p 8080:8080 -p 50000:50000 -v /home/ubuntu/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -d --name jenkins jenkins/jenkins:lts
p.s. In above command both volume and socket are mounted

Running docker in Jenkins container 
https://tutorials.releaseworksacademy.com/learn/the-simple-way-to-run-docker-in-docker-for-ci

