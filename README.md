Example Voting App
=========

Getting started
---------------

Download [Docker](https://www.docker.com/products/overview). If you are on Mac or Windows, [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/). If you're using [Docker for Windows](https://docs.docker.com/docker-for-windows/) on Windows 10 pro or later, you must also [switch to Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).

Run in this directory:
```
docker-compose up
```
The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:
```
docker swarm init
```
Once you have your swarm, in this directory run:
```
docker stack deploy --compose-file docker-stack.yml vote
```

Architecture
-----

![Architecture diagram](architecture.png)

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A .NET worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time


```

[root@bharath1 example-voting-app]# docker stack  deploy --compose-file docker-stack.yml vote
Creating network vote_default
Creating network vote_frontend
Creating network vote_backend
Creating service vote_worker
Creating service vote_visualizer
Creating service vote_redis
Creating service vote_db
Creating service vote_vote
Creating service vote_result
[root@bharath1 example-voting-app]# 

[root@bharath1 example-voting-app]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                                          PORTS
3cm7xq7ixd0p        bharathweb          replicated          12/12               nigelpoulton/tu-demo:v2                        *:9990->80/tcp
eku8zey7zyhd        bkping              replicated          9/9                 alpine
4hlvvzcrsunx        hanuman_ping        replicated          10/10               alpine
64xp9bnk62dy        hanumans            replicated          5/5                 nigelpoulton/pluralsight-docker-ci             *:80->8080/tcp
d44gucr3agll        rajuping            replicated          15/15               alpine
x7rs152di3qa        vote_db             replicated          0/1                 postgres:9.4
lg9goskjjzlo        vote_redis          replicated          1/1                 redis:alpine                                   *:30000->6379/tcp
eg8ozd9v99ap        vote_result         replicated          0/1                 dockersamples/examplevotingapp_result:before   *:5001->80/tcp
pwldq04eo9hp        vote_visualizer     replicated          1/1                 dockersamples/visualizer:stable                *:8080->8080/tcp
r11w2ikd4ean        vote_vote           replicated          1/2                 dockersamples/examplevotingapp_vote:before     *:5000->80/tcp
a5jg52xjij7x        vote_worker         replicated          0/1                 dockersamples/examplevotingapp_worker:latest
[root@bharath1 example-voting-app]#


[root@bharath1 example-voting-app]# docker stack ls
NAME                SERVICES
vote                6
[root@bharath1 example-voting-app]# docker stack ps vote
ID                  NAME                IMAGE                                          NODE                  DESIRED STATE       CURRENT STATE           ERROR               PORTS
c2zud6lewx35        vote_result.1       dockersamples/examplevotingapp_result:before   bharath6.docker.com   Running             Running 3 minutes ago
1nwqi5ps273c        vote_vote.1         dockersamples/examplevotingapp_vote:before     bharath6.docker.com   Running             Running 3 minutes ago
l65hcwe3k0d4        vote_db.1           postgres:9.4                                   bharath3.docker.com   Running             Running 3 minutes ago
te43h5z1r7cx        vote_redis.1        redis:alpine                                   bharath6.docker.com   Running             Running 3 minutes ago
0467eqnapwv8        vote_visualizer.1   dockersamples/visualizer:stable                bharath2.docker.com   Running             Running 3 minutes ago
s7289b0tdf4c        vote_worker.1       dockersamples/examplevotingapp_worker:latest   bharath3.docker.com   Running             Running 3 minutes ago
pjyrfjlbdthr        vote_vote.2         dockersamples/examplevotingapp_vote:before     bharath5.docker.com   Running             Running 3 minutes ago


```



Note
----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.
