---
title: "Docker"
authors: ["gio"]
---

#### Installing docker on Ubuntu

```
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce docker-compose -y
sudo systemctl status docker
```

#### Setup docker swarm nodes

*First node*
```
docker swarm init --advertise-addr <ADDR>
docker swarm join-token (worker|manager)
```

*Rest of the nodes*
Run the instruction specified after running `join-token` command

*Set labels*
```
docker node update --label-add <key>=<value> <node-id>
```

*Deploying*
```
docker stack deploy --compose-file=docker-compose.yml stack-name
```

#### Useful docker commands

See [here](https://towardsdatascience.com/15-docker-commands-you-should-know-970ea5203421)
