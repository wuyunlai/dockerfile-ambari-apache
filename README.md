# dockerfile-ambari-apache
dockerfile-ambari-apache

# From
https://cwiki.apache.org/confluence/display/AMBARI/Development+in+Docker
https://github.com/apache/ambari/tree/trunk/dev-support/docker

#Create Docker Image

First thing first, we have to build an Docker image for this solution. This will setup libraries including ones from yum and maven dependencies. In my environment (Centos 6.5 VM with 8GB and 4CPUs) takes 30mins. Good news is this is one time.
git clone https://github.com/apache/ambari.git
cd ambari
docker build -t ambari/build ./dev-support/docker/docker

This is going to build a image named "ambari/build" from configuration files under ./dev-support/docker/docker

#Unit Test

For example our unit test Jenkins job on trunk runs on Docker. If you want to replicate the environment, read this section.

The basic
cd {ambari_root}
docker run --privileged -h node1.mydomain.com -v $(pwd):/tmp/ambari ambari/build /tmp/ambari/dev-support/docker/docker/bin/ambaribuild.py test -b

    'docker run' is the command to run a container from a image. Which image was run? It is 'ambari/build'
    -h sets a host name in the container. 
    -v is to mount your Ambari code on the host to the container's /tmp. Make sure you are at the Ambari root directory.
    ambaribuild.py runs some script to eventually run 'mvn test' for ambari. 
    -b option is to rebuild the entire source tree. It runs test as is on your host if omitted. 
    
#Deploy Hadoop

You want to run Ambari and Hadoop to test your improvements that you have just coded on your host. Here is the way!

 
cd {ambari_root}
docker run --privileged -t -p 80:80 -p 5005:5005 -p 8080:8080 -h node1.mydomain.com --name ambari1 -v $(pwd):/tmp/ambari ambari/build /tmp/ambari-build-docker/bin/ambaribuild.py deploy -b
  
# once your are done
docker kill ambari1 && docker rm ambari1

    --privileged is important as ambari-server accessing to /proc/??/exe
    -p 80:80 to ensure you can access to web UI from your host.
    -p 5005 is java debug port
    'deploy' to build, install rpms and start ambari-server and ambari-agent and deploy Hadoop through blueprint.

You can take a look at https://github.com/apache/ambari/tree/trunk/dev-support/docker/docker/blueprints to see what is actually deployed.

 

There are a few other parameters you can play.
cd {ambari_root}
docker run --privileged -t -p 80:80 -p 5005:5005 -p 8080:8080 -h node1.mydomain.com --name ambari1 -v ${AMBARI_SRC:-$(pwd)}:/tmp/ambari ambari/build /tmp/ambari-build-docker/bin/ambaribuild.py [test|server|agent|deploy] [-b] [-s [HDP|BIGTOP|PHD]]

    test: mvn test

    server: install and run ambari-server

    agent: install and run ambari-server and ambari-agent

    deploy: install and run ambari-server and ambari-agent, and deploy a hadoop

 
