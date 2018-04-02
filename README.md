# Alastria DNA

Decentraliced network administration for Alastria platform.


## What is for

Alastria-DNA is an application designed on the blockchain platform of Hyperledger Fabric.

This application is designed to decentrally distribute LUA code within a ChainCode, which will be interpreted in the destination node. Subsequently, it may be installed by the administrator within that node after acceptance by each of the destination nodes.

# How is works
Alastria DNA use Hyperledger Fabric with dockers for the distributed legder solution. It's compose of tree main components all of them running in docker containers:

-   Fabric CA (Certificate Authority), allow you to generate certificates per users, every single operations must be sign with a certificate.
-   Fabric Peers, is the place where the ledger is store. One peer maybe part of many channels and every channel is inside the peer. So the peers its endorse and manage the ledger.
-   Fabric Orderer,  the ordering service, it’s the heart of hyperledger, before anything to be commited it must pass the orderer, the main role is to provide order in operations.

Alastria DNA has one orderer of type "solo" (only use for development, in production maybe you want to use with "kafka"), three members: one admin and two peers, all of them are part of one channel (is a private subnet for two o more members).

The are three organizations, which manage the members admin, peer1, peer2.

The organizations manage the members under a single MSP (Membership Service Provider). For Org1 we have Org1MSP, for Org2 we have Org2MSP and for coreAdm we have coreAdmMSP.

In each member we execute a REST API using the Beego framework with the Hyperledger Fabric Client SDK, used to interact with a Hyperledger Fabric blockchain. Then we connect a FrontEnd written in NodeJS and angularJs in the client (References can be found later).

## Limitations
 - This is a WIP prototype which is defined on a fixed quorum network, this is, a network of 2 validator nodes and an admin node. So does in the hyperledger fabric layer, we have 3 nodes in the same channel in the hyperledger fabric. If a new node is joined in this network, currently new hyperledger fabric crypto materials should be generated and a new network has to be definied and restarted (However we are wokring on an automatically hot node installation solution).

 - This demo has not a login process and a secure accessing control.


## Install prerequisites

- Docker and Docker Compose
- Go version 1.9.x [install](https://golang.org/doc/install)
- Npm and Node.js [install](https://docs.npmjs.com/getting-started/installing-node)
- npm install -g gulp-cli bower

``` bash
go get github.com/hyperledger/fabric
go get github.com/hyperledger/fabric-sdk-go
```

Hyperledger fabric is an _Active_ Hyperledger project. Information on what _Active_ entails can be found in
the [Hyperledger Project Lifecycle document](https://wiki.hyperledger.org/community/project-lifecycle).

So Make sure you are working on the same release/commits

- Release Hyperledger Fabric v1.0.6
- Hyperledger Fabric Client SDK for Go v1.0.0-alpha2

``` bash
cd $GOPATH/src/github.com/hyperledger/fabric
git checkout 70f3f2
cd $GOPATH/src/github.com/hyperledger/fabric-sdk-go
git checkout b0efb7
```

If you have Alastria-Dna in other directory, then you shoud link it into $GOPATH/src directory:

``` bash
ln -s [Your path]/alastria-dna $GOPATH/src/github/
ln -s [Your path]/alastria-dna/hyperapi $GOPATH/src/
```

## Run the Test Network

The testnet archicture is made up by 3 organizations with one peer each organization. All the peers belongs to the same channel called "channel"
To run the testnet in development mode execute the following command:

``` bash
cd ./hf-testnet
./fabricOps.sh start
```

## Run Dna Alastria FrontEnd for the admin

``` bash
cd ./dna-frontAdmin
npm install
bower install
gulp serve
```

For more information about Admin console please click [here](https://github.com/alastria/alastria-dna/blob/develop/dna-frontAdmin/README.md).

## Run Dna Alastria FrontEnd for the entity

You need to clone the folder if you need more entities

``` bash
cd ./dna-frontEntity
npm install
bower install
gulp serve
```

For more information about Entity console please click [here](https://github.com/alastria/alastria-dna/blob/develop/dna-frontEntity/README.md).

## Run the API Dna Alastria [Backend](hyperapi/README.md)

API Backend is a middleware to exchange information from front-end to hyperledger fabric chaincode and vice versa. To run the api backend for the whole infrastructure, we need to:

- terminal 1$ ```cd hyperapi && go run main.go conf/coreAdm.conf```
- terminal 2$ ```cd hyperapi && go run main.go conf/entity1.conf```
- terminal 3$ ```cd hyperapi && go run main.go conf/entity2.conf```

For the first time running the application you have to run the server with ```createChaincodeFirstTime()``` and ```createChaincodeLuaExecutorFirstTime()``` uncommented in order to install all the chaincodes we need to run this demo

## Alastria DNA Distributed (example)

To deploy the network in a distributed way we will need 3 EC2 machines.

- admin
- node 1 (Org1)
- node 2 (Org2)

**hyperapi / conf-distributed /**

In the folder "hyperapi / conf-distributed /" we have an example of configuration files for the machines, in which the IP of each machine has been modified.

**hf-testnet / base-distributed**

In the base-distributed folder you can find the files docker-compose-base.yml and peer.yml with the extra_host and the IPs assigned to the peers.

**hf-testnet/crypto-config-distributed**

Here lies the Crypto Material generated for this example.

**hf-testnet/docker-compose-distributed.yaml**

This file is what we use to generate the containers.

**hf-testnet/fabricOp-distributed.sh**

In this script the following functions of *start* and *clean* have been commented for the example:

- #generateCerts
- #generateChannelArtifacts
- #replacePrivateKey
- #startNetwork
- And the *clean* function now only removes the containers, leaving the docker images intact.
  In this way we do not rebuild our crypto materials.


With all this and assuming that we have the machines and the source code and all the dependencies installed in each machine, we proceed with the creation of the Hyperledger network.

### 1. In the admin ec2 machine

``` bash
    docker-compose -f docker-compose.yaml up -d caCoreAdm orderer.alastria.com peer0.coreAdm.alastria.com
```

### 2. In the node1 ec2 machine

``` bash
    docker-compose -f docker-compose.yaml up -d caOrg1 peer0.org1.alastria.com
```

### 3. In the node2 ec2 machine

``` bash
    docker-compose -f docker-compose.yaml up -d caOrg2 peer0.org2.alastria.com
```

### 4. Run the cli container in the admin ec2

``` bash
    docker-compose -f docker-compose.yaml up cli
```

This 4 steps create the whole hf-testnet distributed.


## Upload the LUA Chaincode (for the monitor)

 A query for a rest service written in Lua Language. This piece of code make a query to the [Alastria's Monitor](https://github.com/alastria/monitor).
```
function execute()
  return ServiceCall('https://172.18.0.14:8443/v1/monitor/status', 'GET')
end
```