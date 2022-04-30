---
layout: default
title: Developer Installation
nav_order: 2
parent: Getting Started
---
# Developer Installation

SirMordred is the tool used to coordinate the execution of the GrimoireLab platform, via two main configuration files, the [setup.cfg](./setup-cfg.md) and [projects.json](./projects-json.md), which are summarized in their corresponding sections.

SirModred relies on ElasticSearch, Kibiter and MySQL/MariaDB. The current versions used are:

- ElasticSearch 6.8.6
- Kibiter 6.8.6
- MySQL/MariaDB (5.7.24/10.0)

There are mainly 2 options to get started with SirMordred:
- [Source code and docker](#source-code-and-docker-):
In this method, the applications (ElasticSearch, Kibiter and MariaDB) are installed using docker and the GrimoireLab Components are installed using the source code.
- [Only docker](#only-docker-):
In this method, the applications (ElasticSearch, Kibiter and MariaDB) and the GrimoireLab Components are installed using docker.

## Source code and docker

### Getting the containers

You will have to install ElasticSearch (6.8.6), Kibiter (6.8.6) and a MySQL/MariaDB database (5.7.24/10.0). You can use the following docker-compose to have them running.

> Help: You need to install docker and docker-compose for this. Please refer the documentation.
> - https://docs.docker.com/install/linux/docker-ce/ubuntu/
> - https://docs.docker.com/compose/install/
> Note: 
> 1. You can omit (comment/remove) the `mariadb` section in case you have MariaDB or MySQL already installed in your system.
> 2. It is not mandatory to use docker to install ElasticSearch, Kibiter and MySQL/MariaDB database. They can be installed by other means too (source code). We are not much concerned about the method they are installed. Docker is the easiest way as it mostly avoids the errors caused by them.

**docker-compose (with SearchGuard)**

> **Note**: For accessing Kibiter and/or creating indexes login is required, the `username:password` is `admin:admin` in [`setup.cfg`](https://github.com/chaoss/grimoirelab-sirmordred/blob/master/sirmordred/utils/setup.cfg) file.

```
elasticsearch:
  image: bitergia/elasticsearch:6.8.6-secured
  command: elasticsearch -Enetwork.bind_host=0.0.0.0 -Ehttp.max_content_length=2000mb
  ports:
    - 9200:9200
  environment:
    - ES_JAVA_OPTS=-Xms2g -Xmx2g
kibiter:
  restart: on-failure:5
  image: bitergia/kibiter:secured-v6.8.6-3
  environment:
    - PROJECT_NAME=Demo
    - NODE_OPTIONS=--max-old-space-size=1000
    - ELASTICSEARCH_USER=kibanaserver
    - ELASTICSEARCH_PASSWORD=kibanaserver
    - ELASTICSEARCH_URL=["https://elasticsearch:9200"]
    - LOGIN_SUBTITLE=If you have forgotten your username or password ...
  links:
    - elasticsearch
  ports:
    - 5601:5601
```

 **docker-compose (without SearchGuard)**

> **Note**: Here, access to kibiter and elasticsearch don't need credentials.

```

elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch-oss:6.8.6
  command: elasticsearch -Enetwork.bind_host=0.0.0.0 -Ehttp.max_content_length=2000mb
  ports:
    - 9200:9200
  environment:
    - ES_JAVA_OPTS=-Xms2g -Xmx2g
    - ANONYMOUS_USER=true
kibiter:
  restart: on-failure:5
  image: bitergia/kibiter:community-v6.8.6-3
  environment:
    - PROJECT_NAME=Demo
    - NODE_OPTIONS=--max-old-space-size=1000
    - ELASTICSEARCH_URL=http://elasticsearch:9200
  links:
    - elasticsearch
  ports:
    - 5601:5601
```

Save the above into a docker-compose.yml file and run
```
$ docker-compose up -d
```
to get ElasticSearch, Kibiter and MariaDB running on your system.

### Cloning the repositories

In the next step, you will need to fork all the GitHub repos below and clone them to a target local folder (e.g., `sources`).

- [SirModred](https://github.com/chaoss/grimoirelab-sirmordred)
- [ELK](https://github.com/chaoss/grimoirelab-elk)
- [Graal](https://github.com/chaoss/grimoirelab-graal)
- [Perceval](https://github.com/chaoss/grimoirelab-perceval)
- [Perceval for Mozilla](https://github.com/chaoss/grimoirelab-perceval-mozilla)
- [Perceval for OPNFV](https://github.com/chaoss/grimoirelab-perceval-opnfv)
- [Perceval for Puppet](https://github.com/chaoss/grimoirelab-perceval-puppet)
- [Perceval for Weblate](https://github.com/chaoss/grimoirelab-perceval-weblate)
- [SortingHat](https://github.com/chaoss/grimoirelab-sortinghat)
- [Sigils](https://github.com/chaoss/grimoirelab-sigils)
- [Kidash](https://github.com/chaoss/grimoirelab-kidash)
- [Toolkit](https://github.com/chaoss/grimoirelab-toolkit)
- [Cereslib](https://github.com/chaoss/grimoirelab-cereslib)
- [Manuscripts](https://github.com/chaoss/grimoirelab-manuscripts)

Each local repo should have two `remotes`: `origin` points to the forked repo, while `upstream` points to the original CHAOSS repo.

An example is provided below.
```
$ git remote -v
origin	https://github.com/valeriocos/perceval (fetch)
origin	https://github.com/valeriocos/perceval (push)
upstream	https://github.com/chaoss/grimoirelab-perceval (fetch)
upstream	https://github.com/chaoss/grimoirelab-perceval (push)
```

In order to add a remote to a Git repository, you can use the following command:
```
$ git remote add upstream https://github.com/chaoss/grimoirelab-perceval
```

#### ProTip

You can use this use this [script](https://gist.github.com/vchrombie/4403193198cd79e7ee0079259311f6e8) to automate this whole process.
```
$ python3 glab-dev-env-setup.py --create --token xxxx --source sources
```

### Setting up PyCharm

> Help: 
> You need to install PyCharm (**Community Edition**) for this. Please refer the documentation.
> - https://www.jetbrains.com/help/pycharm/installation-guide.html
>
> You can follow this [tutorial](https://www.jetbrains.com/help/pycharm/quick-start-guide.html) to get familiar with PyCharm.
Once PyCharm is installed create a project in the grimoirelab-sirmordred directory. 
PyCharm will automatically create a virtual env, where you should install the dependencies listed in each 
requirements.txt, **excluding** the ones concerning the grimoirelab components.

To install the dependencies, you can click on `File` -> `Settings` -> `Project` -> `Project Interpreter`, and then the `+` located on the top right corner (see figure below).

![project-interpreter-configuration](https://user-images.githubusercontent.com/25265451/78168870-3e612580-746e-11ea-9df1-7ba94b84d07b.gif)

Later, you can add the dependencies to the grimoirelab components via `File` -> `Settings` -> `Project` -> `Project Structure`. 
The final results should be something similar to the image below.

![project-structure-configuration](https://user-images.githubusercontent.com/25265451/78168879-41f4ac80-746e-11ea-9e40-dbdb1b5d32f2.gif)

### Execution

Now that you have the ElasticSearch, Kibiter and MariaDB running on your system and the project configured in the PyCharm, we can execute micro-mordred/sirmordred. 

To execute micro-mordred, define a [setup.cfg](https://github.com/chaoss/grimoirelab-sirmordred/blob/master/sirmordred/utils/setup.cfg) and [projects.json](https://github.com/chaoss/grimoirelab-sirmordred/blob/master/sirmordred/utils/projects.json), and
run the following commands, which will collect and enrich the data coming from the git sections and upload the corresponding panels to Kibiter:
```
micro.py --raw --enrich --cfg ./setup.cfg --backends git cocom
micro.py --panels --cfg ./setup.cfg
```

Optionally, you can create a configuration in PyCharm to speed up the executions (`Run` -> `Edit configuration` -> `+`).

![add-micro-configuration](https://user-images.githubusercontent.com/25265451/78168875-402ae900-746e-11ea-8bd8-4b3e68992bdf.gif)

The final results should be something similar to the image below.

![result](https://user-images.githubusercontent.com/25265451/84477839-ee90ad00-acad-11ea-932f-cc7ce81e05a7.png)

## Only docker

Follow the instruction in the GrimoireLab tutorial to have [SirMordred in a container](../../sirmordred/container.md)

---