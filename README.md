
Sarmad Kubba
CNE370
# Project Title
Real World Project: Database Shard GitHub.
## Getting Started
This project is how to architecture backend servers and shard the Data with two servers using Maxscale server and show the client he is accessing one server but technically he is accessing two master MariaDB servers connecting with maxscale.

### Prerequisites
The prerequisites for this project is you need to have virtual machine running with OS ubuntu 22.04.1, and you need to install docker and docker-compose.

This is how to install docker community edition.

sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
sudo apt install docker-ce

Check the Docker status with command line.
sudo systemctl run docker
sudo systenctl enable docker
sudo systemctl status docker.

Installation Docker Compose:
sudo apt install docker-compose.

Install client access to MariaDB 
sudo apt install MariaDB-client.
Set up and clone maxscale-docker repository from zohan GitHub.
git clone https://github.com/Zohan/maxscale-docker.git
to run the docker-compose up you must be in right directory "maxscale-docker/maxscale"
docker-compose up -d
check all servers states and use this command.
root@Maxscale:/home/sarmad/maxscale/maxscale-docker/maxscale# docker-compose up -d
Starting maxscale_master1_1 ... done

Starting maxscale_master2_1 ... done

Starting maxscale_maxscale_1 ... done
When the machine is done use the command to check the status of the servers 
root@Maxscale:/home/sarmad/maxscale/maxscale-docker/maxscale# docker ps -a

CONTAINER ID   IMAGE                     COMMAND                  CREATED        STATUS        PORTS                                                                                            NAMES

77b6b90982dc   mariadb/maxscale:latest   "/usr/bin/tini -- do…"   46 hours ago   Up 46 hours   0.0.0.0:4000->4000/tcp, :::4000->4000/tcp, 3306/tcp, 0.0.0.0:8989->8989/tcp, :::8989->8989/tcp   maxscale_maxscale_1

0d917934af7f   mariadb:latest            "docker-entrypoint.s…"   46 hours ago   Up 46 hours   0.0.0.0:4001->3306/tcp, :::4001->3306/tcp                                                        maxscale_master1_1

ffd8334a52f4   mariadb:latest            "docker-entrypoint.s…"   46 hours ago   Up 46 hours   0.0.0.0:4003->3306/tcp, :::4003->3306/tcp                                                        maxscale_master2_1
![image](https://user-images.githubusercontent.com/83927120/227092347-ec29cc6b-7aa5-46ef-a137-83c8a8747951.png)

Check if the servers are running and connecting the ports, use this command.
root@Maxscale:/home/sarmad/maxscale/maxscale-docker/maxscale# docker-compose exec maxscale maxctrl list servers
─────────┬─────────┬──────┬─────────────┬─────────────────┬──────┬─────────────────┐

│ Server  │ Address │ Port │ Connections │ State           │ GTID │ Monitor         │

├─────────┼─────────┼──────┼─────────────┼─────────────────┼──────┼─────────────────┤

│ server1 │ master1 │ 3306 │ 0           │ Running         │      │ MariaDB-Monitor │

├─────────┼─────────┼──────┼─────────────┼─────────────────┼──────┼─────────────────┤

│ server2 │ master2 │ 3306 │ 0           │ Master, Running │      │ MariaDB-Monitor │

└─────────┴─────────┴──────┴─────────────┴─────────────────┴──────┴─────────────────┘
![image](https://user-images.githubusercontent.com/83927120/227092595-4f69ce59-d57a-4a74-b4e0-355bdd94ef25.png)

check if master down how to master2 become master1 and use this command. 
docker-compose stop master.
 root@Maxscale:/home/sarmad/maxscale/maxscale-docker/maxscale# docker-compose stop master1
Stopping maxscale_master1_1 ... done
root@Maxscale:/home/sarmad/maxscale/maxscale-docker/maxscale# docker-compose exec maxscale maxctrl list servers


─────────┬─────────┬──────┬─────────────┬─────────────────┬──────┬─────────────────┐

│ Server  │ Address │ Port │ Connections │ State           │ GTID │ Monitor         │

├─────────┼─────────┼──────┼─────────────┼─────────────────┼──────┼─────────────────┤

│ server1 │ master1 │ 3306 │ 0           │ Down            │      │ MariaDB-Monitor │

├─────────┼─────────┼──────┼─────────────┼─────────────────┼──────┼─────────────────┤

│ server2 │ master2 │ 3306 │ 0           │ Master, Running │      │ MariaDB-Monitor │

└─────────┴─────────┴──────┴─────────────┴─────────────────┴──────┴─────────────────┘
![image](https://user-images.githubusercontent.com/83927120/227092744-c144b62a-f38b-46e6-bb0c-112d0e310ba5.png)

Same with Master2 we can see how we can be down :
root@Maxscale:/home/sarmad/maxscale/maxscale-docker/maxscale# docker-compose stop master2
┌─────────┬─────────┬──────┬─────────────┬─────────────────┬──────┬─────────────────┐

│ Server  │ Address │ Port │ Connections │ State           │ GTID │ Monitor         │

├─────────┼─────────┼──────┼─────────────┼─────────────────┼──────┼─────────────────┤

│ server1 │ master1 │ 3306 │ 0           │ Master, Running │      │ MariaDB-Monitor │

├─────────┼─────────┼──────┼─────────────┼─────────────────┼──────┼─────────────────┤

│ server2 │ master2 │ 3306 │ 0           │ Down            │      │ MariaDB-Monitor │

└─────────┴─────────┴──────┴─────────────┴─────────────────┴──────┴─────────────────┘
![image](https://user-images.githubusercontent.com/83927120/227092881-4c6e020f-5d5c-4801-acc4-e8846e3f1f11.png)

Now create client to access the databases.using this commands
This is the way you can access your data from mysql console.
oot@Maxscale:/home/sarmad/maxscale/maxscale-docker/maxscale# mysql -umaxuser -pmaxpwd -h 127.0.0.1 -P 4000

Welcome to the MariaDB monitor.  Commands end with ; or \g.

Your MariaDB connection id is 1

Server version: 10.11.2-MariaDB-1:10.11.2+maria~ubu2204-log mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
![image](https://user-images.githubusercontent.com/83927120/227093070-a89fbd78-1dc5-4ee5-9c79-d3a8f6d95eef.png)

Now we can use this command to show the databases in our server
MariaDB [(none)]> show databases;

+--------------------+

| Database           |

+--------------------+

| information_schema |

| mysql              |

| performance_schema |

| sys                |

| zipcodes_one       |

| zipcodes_two       |

+--------------------+

6 rows in set (0.001 sec)
![image](https://user-images.githubusercontent.com/83927120/227093225-bdff938b-5aa3-482d-97db-0731a425efda.png)

I can use command use to access both servers from my maxscale server
![image](https://user-images.githubusercontent.com/83927120/227093384-949d3bac-780f-4542-b31a-84ee93030235.png)

## Using python to access remotely our maxscale server 

Running python
This is the output when I run python zipcodes.py


The last 10 rows of zipcodes_one are:
(40843, 'STANDARD', 'HOLMES MILL', 'KY', 'PRIMARY', '36.86', '-83', 'NA-US-KY-HOLMES MILL', 'FALSE', '', '', '')
(41425, 'STANDARD', 'EZEL', 'KY', 'PRIMARY', '37.89', '-83.44', 'NA-US-KY-EZEL', 'FALSE', '390', '801', '10204009')
(40118, 'STANDARD', 'FAIRDALE', 'KY', 'PRIMARY', '38.11', '-85.75', 'NA-US-KY-FAIRDALE', 'FALSE', '4398', '7635', '122449930')
(40020, 'PO BOX', 'FAIRFIELD', 'KY', 'PRIMARY', '37.93', '-85.38', 'NA-US-KY-FAIRFIELD', 'FALSE', '', '', '')
(42221, 'PO BOX', 'FAIRVIEW', 'KY', 'PRIMARY', '36.84', '-87.31', 'NA-US-KY-FAIRVIEW', 'FALSE', '', '', '')
(41426, 'PO BOX', 'FALCON', 'KY', 'PRIMARY', '37.78', '-83', 'NA-US-KY-FALCON', 'FALSE', '', '', '')
(40932, 'PO BOX', 'FALL ROCK', 'KY', 'PRIMARY', '37.22', '-83.78', 'NA-US-KY-FALL ROCK', 'FALSE', '', '', '')
(40119, 'STANDARD', 'FALLS OF ROUGH', 'KY', 'PRIMARY', '37.6', '-86.55', 'NA-US-KY-FALLS OF ROUGH', 'FALSE', '760', '1468', '20771670')
(42039, 'STANDARD', 'FANCY FARM', 'KY', 'PRIMARY', '36.75', '-88.79', 'NA-US-KY-FANCY FARM', 'FALSE', '696', '1317', '20643485')
(40319, 'PO BOX', 'FARMERS', 'KY', 'PRIMARY', '38.14', '-83.54', 'NA-US-KY-FARMERS', 'FALSE', '', '', '')
![image](https://user-images.githubusercontent.com/83927120/227094060-3c31f15f-407a-40de-ad5d-c9c0ac1b98ce.png)

The first 10 rows of zipcodes_two are:
(42040, 'STANDARD', 'FARMINGTON', 'KY', 'PRIMARY', '36.67', '-88.53', 'NA-US-KY-FARMINGTON', 'FALSE', '465', '896', '11562973')
(41524, 'STANDARD', 'FEDSCREEK', 'KY', 'PRIMARY', '37.4', '-82.24', 'NA-US-KY-FEDSCREEK', 'FALSE', '', '', '')
(42533, 'STANDARD', 'FERGUSON', 'KY', 'PRIMARY', '37.06', '-84.59', 'NA-US-KY-FERGUSON', 'FALSE', '429', '761', '9555412')
(40022, 'STANDARD', 'FINCHVILLE', 'KY', 'PRIMARY', '38.15', '-85.31', 'NA-US-KY-FINCHVILLE', 'FALSE', '437', '839', '19909942')
(40023, 'STANDARD', 'FISHERVILLE', 'KY', 'PRIMARY', '38.16', '-85.42', 'NA-US-KY-FISHERVILLE', 'FALSE', '1884', '3733', '113020684')
(41743, 'PO BOX', 'FISTY', 'KY', 'PRIMARY', '37.33', '-83.1', 'NA-US-KY-FISTY', 'FALSE', '', '', '')
(41219, 'STANDARD', 'FLATGAP', 'KY', 'PRIMARY', '37.93', '-82.88', 'NA-US-KY-FLATGAP', 'FALSE', '708', '1397', '20395667')
(40935, 'STANDARD', 'FLAT LICK', 'KY', 'PRIMARY', '36.82', '-83.76', 'NA-US-KY-FLAT LICK', 'FALSE', '752', '1477', '14267237')
(40997, 'STANDARD', 'WALKER', 'KY', 'PRIMARY', '36.88', '-83.71', 'NA-US-KY-WALKER', 'FALSE', '', '', '')
(41139, 'STANDARD', 'FLATWOODS', 'KY', 'PRIMARY', '38.51', '-82.72', 'NA-US-KY-FLATWOODS', 'FALSE', '3692', '6748', '121902277')
![image](https://user-images.githubusercontent.com/83927120/227094157-31219c99-9f11-4727-8b3b-a4b6f1c95f25.png)

The largest zip code number in zipcodes_one is:
(47750,)
![image](https://user-images.githubusercontent.com/83927120/227094223-1b8a6091-3418-421d-b72d-2955f03a9fe4.png)

The smallest zip code number in zipcodes_two is:
(38257,)
![image](https://user-images.githubusercontent.com/83927120/227094354-ae02a412-2a88-4224-af7c-9ffaa1c98910.png)

## Thanks
 I worked this project with our tutor Abdirizak kulmiye.
Sources
https://docs.docker.com/compose/install/
https://mariadb.com/kb/en/mariadb-maxscale-25-simple-sharding-with-two-servers/
https://docs.docker.com/compose/compose-file/
https://github.com/Zohan/maxscale-docker
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04
https://docker-curriculum.com/#what-are-containers-
