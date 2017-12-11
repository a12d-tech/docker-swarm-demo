# Docker Swarm mode (demo)

Demo docker swarm mode.

In this demo we will use 3 nodes to run our cluster.

## Setup:
Create 3 vms with vagrant (and virtualbox as provider)  
Install docker on all the vms
With docker swarm you will always need to declare a node as the cluster manager.
So I named them :
  - leader (Swarm manager)
  - vm01 (node)
  - vm02 (node)

## Vagrant file:
- for multimachine recommended:
  `config.vm.network "private_network", type: "dhcp"`

## Docker swarm mode:
- `vagrant ssh leader`
- should look like this
```
enp0s8  Link encap:Ethernet  HWaddr 08:00:27:3e:ed:0d  
          inet addr:172.28.128.3  Bcast:172.28.128.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe3e:ed0d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:61 errors:0 dropped:0 overruns:0 frame:0
          TX packets:36 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:26586 (26.5 KB)  TX bytes:6396 (6.3 KB)
```
- so node ip is `172.28.128.3`
- `docker swarm init --advertise-addr 172.28.128.3` or `docker swarm init --listen-addr 172.28.128.3`
- output looks like this

      Swarm initialized: current node (yoccyzkppqlmjfyoxafdnk9ii) is now a manager.
      To add a worker to this swarm, run the following command:

      docker swarm join --token SWMTKN-1-4nax7xlzjxuvq33u3ylkclgmib5udy8umvs6zautcnknpugqoa-1ifbfpzc8wvhrpfzl0lh2exn7 172.28.128.3:2377

      To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

- then connect to the other nodes to register them on the manager
- `vagrant ssh vm01`
- run the command `docker swarm join --token [YOUR_TOKEN]`
- output `This node joined a swarm as a worker.`
- repeat the same operation for all your nodes

- Ssh into your Swarm manager and run `docker node ls`
- output
      ubuntu@ubuntu-xenial:~$ docker node ls
      ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
      3031v6u1trgaiihsuy82nxo9k     ubuntu-xenial       Ready               Active              
      uly3dgcky3ham1xdnj6eyc9at     ubuntu-xenial       Ready               Active              
      yoccyzkppqlmjfyoxafdnk9ii *   ubuntu-xenial       Ready               Active              Leader

- Additional commands (`docker node` command only works on the manager node)
- `docker node --help`
- `docker info`

## Running a service on the cluster
- `docker service create -p 80:80 --name webserver nginx`
- `docker service ls`
- `docker service ps webserver`
- you see on which node it is Running
- ssh connect to this node and run `docker ps`, you will see your container running
- `docker service inspect webserver --pretty` to get info
- `docker service scale webserver=5`
- `docker service ps webserver`

## Configure Vms:
- check opened port `netstat -ntlp | grep LISTEN`
- connect to vm `vagrant ssh vm01`
- run `sudo ufw status`
- you should see the opened ports


## Credits
- https://www.youtube.com/watch?v=KC4Ad1DS8xU
- https://www.digitalocean.com/community/tutorials/how-to-create-a-cluster-of-docker-containers-with-docker-swarm-and-digitalocean-on-ubuntu-16-04
- https://www.digitalocean.com/community/tutorials/how-to-configure-the-linux-firewall-for-docker-swarm-on-ubuntu-16-04
