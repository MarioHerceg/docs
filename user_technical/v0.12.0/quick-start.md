## Quick Start

This chapter outlines installing and running Memgraph, as well as executing
basic queries against the database.

### Installation

The Memgraph binary is offered as:

  * Debian package for Debian 9 (Stretch);
  * RPM package for CentOS 7 and
  * Docker image.

After downloading the binary, proceed to the corresponding section below.

NOTE: Currently, newer versions of Memgraph are not backward compatible with
older versions. This is mainly noticeable by unsupported loading of storage
snapshots between different versions.

#### Docker Installation

Before proceeding with the installation, please install the Docker engine on
the system. Instructions on how to install Docker can be found on the
[official Docker website](https://docs.docker.com/engine/installation).
Memgraph Docker image was built with Docker version `1.12` and should be
compatible with all later versions.

After installing and running Docker, download the Memgraph Docker image and
import it with the following command.

```bash
docker load -i /path/to/memgraph-<version>-docker.tar.gz
```

Memgraph is then started with another docker command.

```bash
docker run -p 7687:7687 \
  -v mg_lib:/var/lib/memgraph -v mg_log:/var/log/memgraph -v mg_etc:/etc/memgraph \
  memgraph
```

On success, expect to see output similar to the following.

```bash
Starting 8 workers
Server is fully armed and operational
Listening on 0.0.0.0 at 7687
```

Memgraph is now ready to process queries, you may now proceed to
[querying](#querying). To stop Memgraph, press `Ctrl-c`.

Memgraph configuration is available in Docker's named volume `mg_etc`. On
Linux systems it should be in
`/var/lib/docker/volumes/mg_etc/_data/memgraph.conf`. After changing the
configuration, Memgraph needs to be restarted.

##### Note about named volumes

In case named volumes are reused between different versions of Memgraph, a user
has to be careful because Docker will overwrite a folder within the container
with existing data from the host machine. In the case where a new file is
introduced, or two versions of Memgraph are not compatible, the new feature
won't work or Memgraph won't be able to work correctly. The easiest way to
solve the issue is to use another named volume or to remove existing named
volume from the host with the following command.

```bash
docker volume rm <volume_name>
```

Named Docker volumes used in this documentation are: `mg_etc`, `mg_log` and
`mg_lib`. E.g. to avoid any configuration issues between different Memgraph
versions, `docker volume rm mg_etc` can be executed before running a new
container.

Another valid option is to try to migrate your existing volume to a
newer version of Memgraph. In case of any issues, send an email to
`tech@memgraph.com`.

##### Note for OS X/macOS Users

Although unlikely, some OS X/macOS users might experience minor difficulties
after following the Docker installation instructions. Instead of running on
`localhost`, a Docker container for Memgraph might be running on a custom IP
address. Fortunately, that IP address can be found using the following
algorithm:

1) Find out the container ID of the Memgraph container

By issuing the command `docker ps` the user should get an output similar to the
following:

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED        ...
9397623cd87e        memgraph            "/usr/lib/memgraph/m…"   2 seconds ago  ...
```

At this point, it is important to remember the container ID of the Memgraph
image.  In our case, that is `9397623cd87e`.

2) Use the container ID to retrieve an IP of the container

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 9397623cd87e
```

The command above should yield the sought IP. If that IP does not correspond to
`localhost`, it should be used instead of `localhost` when firing up the
`neo4j-client` in the [querying](#querying) section.

#### Debian Package Installation

After downloading Memgraph as a Debian package, install it by running the
following.

```bash
dpkg -i /path/to/memgraph_<version>.deb
```

If the installation was successful, Memgraph should already be running. To
make sure that is true, start it explicitly with the command:

```bash
systemctl start memgraph
```

To verify that Memgraph is running, run the following command.

```bash
journalctl --unit memgraph
```

It is expected to see something like the following output.

```bash
Nov 23 13:40:13 hostname memgraph[14654]: Starting 8 BoltS workers
Nov 23 13:40:13 hostname memgraph[14654]: BoltS server is fully armed and operational
Nov 23 13:40:13 hostname memgraph[14654]: BoltS listening on 0.0.0.0 at 7687
```

Memgraph is now ready to process queries, you may now proceed to
[querying](#querying). To shutdown Memgraph server, issue the following
command.

```bash
systemctl stop memgraph
```

Memgraph configuration is available in `/etc/memgraph/memgraph.conf`. After
changing the configuration, Memgraph needs to be restarted.

#### RPM Package Installation

If you downloaded the RPM package of Memgraph, you can install it by running
the following command.

```bash
rpm -U /path/to/memgraph-<version>.rpm
```

After the successful installation, Memgraph can be started as a service. To do
so, type the following command.

```bash
systemctl start memgraph
```

To verify that Memgraph is running, run the following command.

```bash
journalctl --unit memgraph
```

It is expected to see something like the following output.

```bash
Nov 23 13:40:13 hostname memgraph[14654]: Starting 8 BoltS workers
Nov 23 13:40:13 hostname memgraph[14654]: BoltS server is fully armed and operational
Nov 23 13:40:13 hostname memgraph[14654]: BoltS listening on 0.0.0.0 at 7687
```

Memgraph is now ready to process queries, you may now proceed to
[querying](#querying). To shutdown Memgraph server, issue the following
command.

```bash
systemctl stop memgraph
```

Memgraph configuration is available in `/etc/memgraph/memgraph.conf`. After
changing the configuration, Memgraph needs to be restarted.

### Querying

Memgraph supports the openCypher query language which has been developed by
[Neo4j](http://neo4j.com). The language is currently going through a
vendor-independent standardization process. It's a declarative language
developed specifically for interaction with graph databases.

The easiest way to execute openCypher queries against Memgraph, is using
Neo4j's command-line tool. The command-line `neo4j-client` can be installed as
described [on the official website](https://neo4j-client.net).

After installing `neo4j-client`, connect to the running Memgraph instance by
issuing the following shell command.

```bash
neo4j-client -u "" -p ""  localhost 7687
```

After the client has started it should present a command prompt similar to:

```bash
neo4j-client 2.1.3
Enter `:help` for usage hints.
Connected to 'neo4j://@localhost:7687'
neo4j>
```

At this point it is possible to execute openCypher queries on Memgraph. Each
query needs to end with the `;` (*semicolon*) character. For example:

```opencypher
CREATE (u:User {name: "Alice"})-[:Likes]->(m:Software {name: "Memgraph"});
```

The above will create 2 nodes in the database, one labeled "User" with name
"Alice" and the other labeled "Software" with name "Memgraph". It will also
create a relationship that "Alice" *likes* "Memgraph".

To find created nodes and relationships, execute the following query:

```opencypher
MATCH (u:User)-[r]->(x) RETURN u, r, x;
```

#### Supported Languages

If users wish to query Memgraph programmatically, they can do so using the
[Bolt protocol](https://boltprotocol.org). Bolt was designed for efficient
communication with graph databases and Memgraph supports
[Version 1](https://boltprotocol.org/v1) of the protocol. Bolt protocol drivers
for some popular programming languages are listed below:

  * [Java](https://github.com/neo4j/neo4j-java-driver)
  * [Python](https://github.com/neo4j/neo4j-python-driver)
  * [JavaScript](https://github.com/neo4j/neo4j-javascript-driver)
  * [C#](https://github.com/neo4j/neo4j-dotnet-driver)
  * [Ruby](https://github.com/neo4jrb/neo4j)
  * [Haskell](https://github.com/zmactep/hasbolt)
  * [PHP](https://github.com/graphaware/neo4j-bolt-php)

We have included some basic usage examples for some of the supported languages
in the [Drivers](drivers.md) section.

### Telemetry

Telemetry is an automated process by which some useful data is collected at
a remote point. At Memgraph, we use telemetry for the sole purpose of improving
our product, thereby collecting some data about the machine that executes the
database (CPU, memory, OS and kernel information) as well as some data about the
database runtime (CPU usage, memory usage, vertices and edges count).

Here at Memgraph, we deeply care about the privacy of our users and do not
collect any sensitive information. If users wish to disable Memgraph's telemetry
features, they can easily do so by either altering the line in
`/etc/memgraph/memgraph.conf` that enables telemetry (`--telemetry-enabled=true`)
into `--telemetry-enabled=false`, or by including the `--telemetry-enabled=false`
as a command-line argument when running the executable.

### Where to Next

To learn more about the openCypher language, visit [openCypher Query
Language](open-cypher.md) chapter in this document. For real-world examples
of how to use Memgraph visit [Examples](examples.md) chapter. Details on
what can be stored in Memgraph are in [Data Storage](storage.md) chapter.

We *welcome and encourage* your feedback!

