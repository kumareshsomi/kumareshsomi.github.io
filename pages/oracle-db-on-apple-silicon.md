<small>Published on: 21 July 2023</small>

# Install Oracle Database on Apple Silicon

Oracle database is available only for x86_64 architecture. If you have Apple MAC with Apple Silicon (M1/M2..), that uses ARM64 architecture and therefore is not compatible with Oracle database.
We can work around that by using an Oracle database container in a VM emulating x86_64.

You need to have brew, docker and qemu. If you do not have it, first get them.
Sometimes, an existing installation of docker desktop might cause problems. So it is best to do this from command line.

##### Install and Update homebrew
~~~
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

brew update
~~~

##### Install Docker
~~~
brew install docker
~~~

##### Install Colima
~~~
brew install colima
~~~

##### Install QEMU
~~~
brew reinstall qemu
~~~

##### Create a x86_64 VM
~~~
colima start --arch x86_64 --memory 4
~~~

##### Create an Oracle DB container
Thanks to [Gerald Venzl](https://hub.docker.com/u/gvenzl) for making various Oracle DB images available from early on. We will be using it in the example below.
You can also choose to use [Oracle’s official image](container-registry.oracle.com/database/free) instead.
~~~
docker run -d -p 1521:1521 -e ORACLE_PASSWORD=password -v oracle-volume:/opt/oracle/oradata gvenzl/oracle-xe
~~~
This will take few minutes.

##### Access the database directly and create your user
For example:
~~~
docker ps

docker exec -it <container id> bash

sqlplus system/password@//localhost/XEPDB1

create user <user name> identified by password default tablespace users;

grant connect, resource to <user name>;

exit

sqlplus <user name> /password@//localhost/XEPDB1

exit
~~~

Now you can connect from your MAC host via sqldeveloper wtih <user name> user, connection type basic, role default, hostnme localhost and service name XEPDB1 (not SID).
You can install sqldevelper for Apple Silicon using [Mac ARM64 with JDK 11 included](https://www.oracle.com/database/sqldeveloper/technologies/download/) .

To stop the oracle DB container and start it again later on:

~~~
colima stop

colima start

docker ps -a

docker start <container id>
~~~

This is one way to get Oracle DB running if you ever need it locally on your new MAC. Nowadays, you could also consider using [testcontainers desktop app](https://testcontainers.com/desktop/) to locally run your tests using Oracle DB.

![Visitor Count](https://visitor-badge.laobi.icu/badge?page_id=kumaresh.github.io.oracle-db-on-apple-silicon)