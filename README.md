# Vlok

## Overview
Vlok is a decentralized service for the generation of unique, time-ordered IDs.

It generates cluster-wide unique IDs with high performance using the [Akka IO](http://doc.akka.io/docs/akka/snapshot/scala/io.html) implementation through [Bark](http://github.com/lab050/bark).

The current implementation is able to push over **1 million ids** per second from server to clients on commodity hardware.

## Why no slowflake?
`Vlok` is Dutch for "Flake" and while we have our cheese, wooden shoes and windmills. We love large numbers ($) and cheap things the most!

The current implementation of [Snowflake](https://github.com/twitter/snowflake) has the requirement to coordinate the uniqueness of IDs through "server ids" generated through Zookeeper. Something as simple as ID generation shouldn't be weighted down by a implementation of Zookeeper (especially if you aren't using it anywhere else).

The default size of Snowflake's IDs are 64 bits wide, sporting a 41 bit timestamp and and 22 bits for the worker id and sequence id (if several ids would be generated at closely the same time).

Because our architecture doesn't have direct limits for a key size of 64 bits, `Vlok` uses 128 bits ids for more detailed timestamps and the possibility of cluster-wide uniqueness without introducing a managing architecture. Resulting in a cheaper usage, using larger numbers!

## Anatomy
128-bits wide described here from most significant to least significant bits.

* 64-bit timestamp - milliseconds since the epoch (Jan 1 1970)
* 48-bit mac address - MAC address from a configurable device
* 16-bit sequence # - usually 0, incremented when more than one id is requested in the same millisecond and reset to 0 when the clock ticks forward

### Formatting
The generated IDs are returned as a `BigInt`, leaving it up to the developer if it will be encoded `Base62` or `Base64` (or any other) in the end. The direct storage of the ids as numbers in NoSQL databases can help immensely with indexing and range queries.

## Usage
The `Vlok` package consists of both a server as a client implementation. The usage of both is very straight forward:
### Server
The server application of `Vlok` can be run using `sbt`:

```scala
sbt "run en0 6000"
```

The application takes a ethernet device as it's first parameter (to ensure it's uniqueness) and the port to run on as its second argument.

### Client
The client can be used by importing the `import nl.gideondk.vlok.client` package and initializing the client to the server running the `Vlok` server application:

```scala
import nl.gideondk.vlok.client._

val clientSystem = ActorSystem("client-system")
val client = VlokClient("localhost", 9999, 32)(clientSystem)
```

New ids can be generated by using the `generateID` function on the client:

```scala
client.generateID
res0: Task[BigInt]
```

## Installation
To run the server application, it's currently best to download the code and running it directly on your server. In the future, the application will be packaged for easier installation and usage.

For usage of the client within applications, there is a repository available through SBT.

Start by adding the repo:
```scala
"gideondk-repo" at "https://raw.github.com/gideondk/gideondk-mvn-repo/master"
```

to your SBT configuration and adding the package to your library dependencies:

```scala
libraryDependencies ++= Seq(
	"nl.gideondk" %% "vlok" % "0.2.2"
)
```

# License
Copyright © 2013 Gideon de Kok

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
