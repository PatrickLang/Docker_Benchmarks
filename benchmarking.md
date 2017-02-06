
1. Got a test dataset - 700MB of files of various sizes from a backup
2. Created 2 VMs running Docker v1.13
 - Windows Server 2016 & Ubuntu 16.04
 - 2Gb memory (fixed)
 - 2 cores. Host has 4 (Intel Core i5-4670s)
3. `docker build`, capture performance on each
4. `docker push` to a registry in a 3rd VMs
5. Reprovision two VMs in step 2 `cmd needed`
6. `docker pull` on each


## Registry Setup

```
docker run -d -p 5000:5000 --restart=always --name registry \
  -v `pwd`/data:/var/lib/registry \
  registry:2
```


## Windows build

Dockerfile:
```
FROM microsoft/windowsservercore
ADD roondata /roondata
```


`Measure-Command { docker build -t roondata:windows . }`

results:
```
Days              : 0
Hours             : 0
Minutes           : 2
Seconds           : 42
Milliseconds      : 870
Ticks             : 1628701201
TotalDays         : 0.00188507083449074
TotalHours        : 0.0452417000277778
TotalMinutes      : 2.71450200166667
TotalSeconds      : 162.8701201
TotalMilliseconds : 162870.1201
```

## Linux build

Dockerfile:
```

```


`time docker build -t roondata:linux .`


Results
```
vagrant@linux1:~/benchmark$ time docker build -t roondata:linux .
Sending build context to Docker daemon 1.009 GB
Step 1/2 : FROM ubuntu
 ---> f49eec89601e
Step 2/2 : ADD roondata /roondata
 ---> 60ff5f1195b3
Removing intermediate container b0462c32a62c
Successfully built 60ff5f1195b3

real    1m0.835s
user    0m1.724s
sys     0m2.124s
```

Push it

Create `/etc/docker/daemon.json`

```
{
  "insecure-registries": [
    "192.168.1.137:5000" ]
}
```

`docker tag 60 192.168.1.137:5000/roondata:linux`
`docker push 192.168.1.137:5000/roondata:linux`

## Reset the VMs
```
vagrant destroy windows
vagrant up windows
vagrant destroy linux1
vagrant up linux1
```



## Time pull


Windows: 
```
[192.168.1.138]: PS C:\Users\vagrant\Documents> measure-command { docker pull 192.168.1.137:5000/roondata:windows }


Days              : 0
Hours             : 0
Minutes           : 1
Seconds           : 23
Milliseconds      : 179
Ticks             : 831790270
TotalDays         : 0.000962720219907407
TotalHours        : 0.0231052852777778
TotalMinutes      : 1.38631711666667
TotalSeconds      : 83.179027
TotalMilliseconds : 83179.027
```

Linux:
```
docker pull ubuntu
time docker pull 192.168.1.137:5000/roondata:linux
```

```
vagrant@linux1:~$ time docker pull 192.168.1.137:5000/roondata:linux
linux: Pulling from roondata
e2d7e96004fd: Pull complete
1bacd3c8ccb1: Pull complete
869d5d3f92f8: Pull complete
f8a4e25b40ce: Pull complete
f40e1388890a: Pull complete
2cca37efc7bc: Pull complete
Digest: sha256:db503286ca720481f55a82b4338869f31a73d7c599157342d1bbc560306a9778
Status: Downloaded newer image for 192.168.1.137:5000/roondata:linux

real    0m40.277s
user    0m0.064s
sys     0m0.036s
```
