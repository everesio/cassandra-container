cassandra-docker
================

Standlone/Clustered Datastax Community running on top of a Ubuntu-based Docker image with opscenter agent

This was started from the docker-cassandra project and enhanced (substantially)

NOTE: Under construction

Running as standalone
=====================

### Docker

Just start the image normally

    docker run -d mikeln/cassandra_mln

The necessary ports are exposed, so you can interact with the docker image as you normally do.

If you need help, see [Usage Guide](#usage-guide).


### Vagrant

Simply use the builtin `docker` provisioner to pull and start the image.

````ruby
config.vm.provision "docker" do |d|
    d.pull_images "mikeln/cassandra_mln"
    d.run "cass", image: "mikeln/cassandra_mln"
end
````


Running as a cluster
====================

To get the images to talk to each other, simply use the docker `link` and `name` options.

We can __link__ up to 10 containers.

You are able to create as many containers as you'd like, but the init script only considers up to the first 10 as seeds.

Truly, you only need one other seed and for simplicity, that's how this example works. The Cassandra cluster will gossip its way into being fully networked rather quickly. There is no reason a machine `cass9999` can't just reuse a seed from `cass[0-9]`.


### Docker

Start a first image, and then link them all up.

##### Start the first image

    docker run -d -name cass0 mikeln/cassandra_mln

##### Link the cluster

    docker run -d -name cass1 -link cass0:cass0 mikeln/cassandra_mln
    docker run -d -name cass2 -link cass0:cass0 -link cass1:cass1 mikeln/cassandra_mln


### Vagrant

We'll do the same as above, but using the `docker` provisioner.

````ruby
config.vm.provision "docker" do |d|
    d.run "seed", auto_assign_name: false,
      args: "--name cass0",
      image: "mikeln/cassandra_mln"

    d.run "first", auto_assign_name: false,
      args: "--name cass1 --link cass0:cass0",
      image: "mikeln/cassandra_mln"

    d.run "second", auto_assign_name: false,
      args: "--name cass2 --link cass0:cass0 --link cass1:cass1",
      image: "mikeln/cassandra_mln"
end
````



Usage Guide
===========

## Basic Docker One-Liners

Get image ID for `cassandra-docker` on your machine

    docker images | grep "mikeln/cassandra_mln" | sed -e "s/\s\+/ /g" | cut -d' ' -f3

__This is a very convenient export: the image ID__

    export CASSDOCK_ID=`docker images | grep "mikeln/cassandra_mln" | sed -e "s/\s\+/ /g" | cut -d' ' -f3`

List the IPs of containers running `cassandra-docker`

    docker inspect $CASSDOCK_ID | grep IPAddress | sed 's/"IPAddress": "/ /g' | sed 's/",//g' | sed 's/ //g'

Get the above in a comma-separated format

    docker inspect $CASSDOCK_ID | grep IPAddress | sed 's/"IPAddress": "/ /g' | sed 's/",//g' | sed 's/ //g' \
    sed -e :a -e N -e 's/\n/,/' -e ta

Or with spaces if you prefer

    docker inspect $CASSDOCK_ID | grep IPAddress | sed 's/"IPAddress": "/ /g' | sed 's/",//g' | sed 's/ //g' \
    sed -e :a -e N -e 's/\n/ /' -e ta


## Talking to Cassandra

Every docker image is given an IP to communicate with.

Use the above commands to get the IP of your docker images. Or manually look them up using `docker ps` and `docker inspect`.

The Following Ports are open (default Cassandra ports):

    7199 7000 7001 9160 9042

Datastax-agent ports:

    61620 61621 50031 (22)

