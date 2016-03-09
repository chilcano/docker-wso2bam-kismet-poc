# Running multi-container Docker Application for 802.11 traffic capture PoC

![802.11 traffic capture PoC - Docker Compose](https://github.com/chilcano/docker-wso2bam-kismet-poc/blob/master/chilcano-wso2bam-cep-siddhi-wifi-kismet-thrift-cassandra-docker-compose.png "802.11 traffic capture PoC - Docker Compose")


## Getting started

__1) Download Docker Compose file, WSO2 BAM Toolbox and Stream definitions__

```bash
$ git clone https://github.com/chilcano/docker-wso2bam-kismet-poc.git
$ cd docker-wso2bam-kismet-poc
$ git clone https://github.com/chilcano/wso2bam-wifi-thrift-cassandra-poc.git
```

__2) Run docker-compose.yml__

```bash
$ docker-compose up -d
Starting dockerwso2bamkismetpoc_mac-manuf_1
Starting dockerwso2bamkismetpoc_wso2bam-dashboard-kismet_1
```

__3) Check the created containers__

```bash
$ docker ps -q | xargs docker inspect --format '{{printf "%.12s\t%s" .Id .Config.Cmd}}'
32650114efbf  [/bin/sh -c sh ./wso2server.sh]
50ef76b399c3  [/bin/sh -c python mac_manuf_api_rest.py]
```
or
```bash
$ docker ps
CONTAINER ID        IMAGE                                             COMMAND                  CREATED             STATUS              PORTS                                                                                     NAMES
32650114efbf        dockerwso2bamkismetpoc_wso2bam-dashboard-kismet   "/bin/sh -c 'sh ./wso"   6 minutes ago       Up 6 minutes        7611/tcp, 9160/tcp, 9763/tcp, 21000/tcp, 0.0.0.0:7713->7711/tcp, 0.0.0.0:9445->9443/tcp   dockerwso2bamkismetpoc_wso2bam-dashboard-kismet_1
50ef76b399c3        chilcano/mac-manuf-lookup:py-latest               "/bin/sh -c 'python m"   7 minutes ago       Up 6 minutes        5000/tcp, 0.0.0.0:5443->5443/tcp                                                          dockerwso2bamkismetpoc_mac-manuf_1
```

__4) Testing the running containers__

```bash
$ docker-compose logs

Attaching to 250kismetpoc_wso2bam-dashboard-kismet_1, 250kismetpoc_mac-manuf_1
wso2bam-dashboard-kismet_1 | JAVA_HOME environment variable is set to /usr
wso2bam-dashboard-kismet_1 | CARBON_HOME environment variable is set to /opt/wso2bam02a
...
wso2bam-dashboard-kismet_1 | [2016-03-08 17:12:01,714]  INFO {org.wso2.carbon.core.services.util.CarbonAuthenticationUtil} -  'admin@carbon.super [-1234]' logged in at [2016-03-08 17:12:01,714+0000]
mac-manuf_1                |  * Running on https://0.0.0.0:5443/ (Press CTRL+C to quit)
mac-manuf_1                |  * Restarting with stat
mac-manuf_1                |  * Debugger is active!
mac-manuf_1                |  * Debugger pin code: 162-079-671
mac-manuf_1                | 192.168.99.1 - - [08/Mar/2016 17:03:46] "GET /chilcano/api/manuf/00-50:Ca-ca-fe-ca HTTP/1.1" 200 -
...
```

Where the `wso2bam-dashboard-kismet_1` and `mac-manuf_1` are the recently created docker containers.


__5) Stopping or killing the recently created containers__

```bash
$ docker-compose stop
Stopping dockerwso2bamkismetpoc_wso2bam-dashboard-kismet_1 ... done
Stopping dockerwso2bamkismetpoc_mac-manuf_1 ... done
```

```bash
$ docker-compose kill
```


## Running the PoC (Kismet -> WSO2 BAM -> MAC Manuf Lookup)

__1) Send 802.11 captured traffic to WSO2 BAM__

From the Host forwards the `7713` port and exposes the Docker machine IP to your local network. Using this, the Raspberry Pi (Kismet) can reach to the Docker machine.
```bash
// (user/pwd: docker/tcuser)
$ ssh docker@$(docker-machine ip default) -f -N -L 192.168.1.43:7713:localhost:7713
```

Where:

- `f` is to run in background.
- `N` is to don't execute a remote command. This is useful for just forwarding ports.
- `L 192.168.1.43:7713:localhost:7713` this specifies that the given port on the local (client) host is to be forwarded to the given host and port on the remote side.

For a further explanation, follow this instruccions:
https://holisticsecurity.wordpress.com/2016/02/02/everything-generates-data-capturing-wifi-anonymous-traffic-raspberrypi-wso2-part-i

__2) Run the docker-componse.yml (WSO2 BAM & MAC Manuf Lookup)__

Follow the above first part of this guide (`Getting started`).

__3) Visualize the 802.11 captured traffic from the WSO2 BAM Kismet Toolbox (Dashboard)__

Open a browser with this URL `https://192.168.99.100:9445/carbon` and go to `BAM Dashboard` section to view the incoming traffic.
The IP address `192.168.99.100` is the IP of your Docker Machine where are running the 2 above containers (WSO2 BAM and MAC Address Manufacturer containers).

![Visualizing 802.11 captured traffic with the MAC Address Manufacturer](https://github.com/chilcano/docker-wso2bam-kismet-poc/blob/master/chilcano-wso2bam-wifi-thrift-cassandra-5-kismet-toolbox-docker-manuf.png "Visualizing 802.11 captured traffic with the MAC Address Manufacturer")


## References

1. Docker Compose: https://docs.docker.com/compose/overview
2. A Python Microservice in a Docker Container (MAC Address MAnufacturer Lookup): http://wp.me/p8pPj-qG 
3. Analyzing Wireless traffic in real time with WSO2 BAM, Apache Cassandra, Complex Event Processor (CEP Siddhi), Apache Thrift and Python: 
  * Part I (http://wp.me/p8pPj-pE)
  * Part II (http://wp.me/p8pPj-pW)
  * Part III (http://wp.me/p8pPj-qe)
