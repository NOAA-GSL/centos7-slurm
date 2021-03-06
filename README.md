# Slurm on CentOS 7 Docker Image

![CI for CentOS7-Slurm](https://github.com/NOAA-GSL/centos7-slurm/workflows/CI%20for%20CentOS7-Slurm/badge.svg?branch=develop)


This is an all-in-one [Slurm](https://slurm.schedmd.com/) installation (originally forked from [giovtorres/docker-centos7-slurm](https://github.com/giovtorres/docker-centos7-slurm)).  This
container runs the following processes:

* slurmd (The compute node daemon for Slurm)
* slurmctld (The central management daemon of Slurm)
* slurmdbd (Slurm database daemon)
* munged (Authentication service for creating and validating credentials)
* mariadb (MySQL compatible database)
* supervisord (A process control system)

It also has the following packages installed

* Python 3.7
* Java 11
* Scala
* Sbt

## Usage

There is currently only one
[tag](https://hub.docker.com/r/noaagsl/centos7-slurm/tags/)
available, but more may follow.  To use the latest available image, run:

```shell
docker pull noaagsl/centos7-slurm:latest
docker run -it -h ernie noaagsl/centos7-slurm:latest
```

The above command will drop you into a bash shell inside the container. Tini
is responsible for `init` and supervisord is the process control system . To
view the status of all the processes, run:

```shell
[root@ernie /]# supervisorctl status
munged                           RUNNING   pid 23, uptime 0:02:35
mysqld                           RUNNING   pid 24, uptime 0:02:35
slurmctld                        RUNNING   pid 25, uptime 0:02:35
slurmd                           RUNNING   pid 22, uptime 0:02:35
slurmdbd                         RUNNING   pid 26, uptime 0:02:35
```

In `slurm.conf`, the **ControlMachine** hostname is set to **ernie**. Since
this is an all-in-one installation, the hostname must match **ControlMachine**.
Therefore, you must pass the `-h ernie` to docker at run time so that the
hostnames match.

You can run the usual slurm commands:

```shell
[root@ernie /]# sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
normal*      up 5-00:00:00      5   idle c[1-5]
debug        up 5-00:00:00      5   idle c[6-10]
```

```shell
[root@ernie /]# scontrol show partition
PartitionName=normal
   AllowGroups=ALL AllowAccounts=ALL AllowQos=ALL
   AllocNodes=ALL Default=YES QoS=N/A
   DefaultTime=5-00:00:00 DisableRootJobs=NO ExclusiveUser=NO GraceTime=0 Hidden=NO
   MaxNodes=1 MaxTime=5-00:00:00 MinNodes=1 LLN=NO MaxCPUsPerNode=UNLIMITED
   Nodes=c[1-5]
   PriorityJobFactor=50 PriorityTier=50 RootOnly=NO ReqResv=NO OverSubscribe=NO PreemptMode=OFF
   State=UP TotalCPUs=5 TotalNodes=5 SelectTypeParameters=NONE
   DefMemPerCPU=500 MaxMemPerNode=UNLIMITED
```

You can also run the container in detached mode:

```shell
docker run -d -t -h ernie --name slurm noaagsl/centos7-slurm:latest
```

The above command will start the Slurm cluster in the container and you can then interact with it:

```shell
$ docker exec slurm sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
normal*      up 5-00:00:00      5   idle c[1-5]
debug        up 5-00:00:00      5   down c[6-10]
```

```shell
$ docker exec slurm sbatch --wrap="sleep 60"
Submitted batch job 2
```

```shell
$ docker exec chiltepin-slurm squeue                  
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
                 2    normal     wrap     root  R       0:05      1 c1
```


## Building

```shell
git clone https://github.com/NOAA-GSL/centos7-slurm.git
docker build -t centos7-slurm .
```

### Using Build Args

At the moment, there are no build arguments available. The original version
from [giovtorres/docker-centos7-slurm](https://github.com/giovtorres/docker-centos7-slurm)
does have several options available, however.

## Using docker-compose

The included docker-compose file will run the cluster container in the
background.  The docker-compose file uses data volumes to store the slurm state
between container runs.  To start the cluster container, run:

```shell
docker-compose up -d
```

To execute commands in the container, use `docker exec`:

```shell
docker exec centos7-slurm sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
normal*      up 5-00:00:00      5   idle c[1-5]
debug        up 5-00:00:00      5   idle c[6-10]

docker exec centos7-slurm sbatch --wrap="sleep 10"
Submitted batch job 27

docker exec dockercentos7slurm_slurm_1 squeue
            JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            27    normal     wrap     root  R       0:07      1 c1
```

To attach to the bash shell inside the running container, run:

```shell
docker attach centos7-slurm
```

Press `Ctrl-p,Ctrl-q` to detach from the container without killing the bash
process and stopping the container.

To stop the cluster container, run:

```shell
docker-compose down
```

## Testing Locally

[Testinfra](https://testinfra.readthedocs.io/en/latest/index.html) is used to
build and run a Docker container test fixture. Run the tests with
[pytest](https://docs.pytest.org/en/latest/):

```shell
pytest -v
```
