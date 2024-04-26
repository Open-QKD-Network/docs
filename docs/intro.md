---
sidebar_position: 2
---

# Quickstart

Let's discover **qkd-net in less than 30 minutes**.

## Getting Started

## What you'll need
> Below we're assuming that you're running Ubuntu.

Please install the following before proceeding and make sure the executables are in system path.

1. Java SDK 8 or later - http://www.oracle.com/technetwork/java/javase/downloads/index.html
Below we show a sample installation of OpenJDK 11.
```bash
sudo apt install openjdk-11-jre-headless
```
2. Maven - https://maven.apache.org/.
```bash
sudo apt install maven
```
3. screen

Check whether you already have this by running `screen -v`. 
If that fails please install it:
```bash
sudo apt install screen
```
4. git
Check whether you already have it by running: `git --version`.
If not, please install it:
```bash
sudo apt install git
```
5. C requirements for testing
We need these for running the **tls-kms-demo** which is written in C. It makes API calls to qkd-net.
`sudo apt install make gcc libcurl4-openssl-dev libssl-dev libjson-c-dev`

### Optional dependencies
1. curl: `sudo apt install curl`
2. jq: `sudo apt install jq`.
3. tree: `sudo apt install tree`.

## Setting up
> We assume that the IP address of **A** is **192.168.2.207** and IP address of **B** is **192.168.2.212**. Please make the appropriate modifications when running these steps.

### Ubuntu A
-  `git clone https://github.com/Open-QKD-Network/qkd-net.git`
- `cd qkd-net`
-  `git checkout disable-spring-auth`
-  `cd kms`
- `./scripts/build`

#### .qkd modification
- The previous step generates `~/.qkd` directory, but we're going to remove that: `rm ~/.qkd`.
- Move up a directory using `cd ..` and run `tar xvf **qkd-kaiduan-a.tar`. This just unzips the file.
- This unzipped file contained the customized`.qkd` for this repo. We're going to move it to the home directory: `mv .qkd ~/`.
- We need to edit to files some files in the `.qkd` direcotry though. cd into via `cd ~/.qkd`. Use Vim (or any text editor of your choice). Add **false** to the end of **BOTH** `kms.conf` and `kms/kms.conf` to disable OAuth authentication. 
- After this both `kms.conf` and `kms/kms.conf` should look like:
```
http://localhost:9992/uaa/oauth/token
http://localhost:8095/api/newkey
http://localhost:8095/api/getkey
A
false
```
- Changes IP address of **B** in the `~/.qkd/qnl/routes.json`.
```json
{
  "adjacent": {
    "B": "192.168.2.212"
  },
  "nonAdjacent": {
  }
}
```

### Ubuntu B
This is going to be very similar to what we did for Ubuntu A.
- `git clone https://github.com/Open-QKD-Network/qkd-net.git`
- `cd qkd-net`
- `git checkout disable-spring-auth`
- `cd kms`
- `./scripts/build`
- `rm ~/.qkd`
- `cd ..`
- `tar xvf qkd-kaiduan-b.tar`
- `mv .qkd ~/`
- Like we did for Ubuntu A, `cd ~/.qkd` and use your favourite text editor to add **false** to the end of **both**`kms.conf` and `kms/kms.conf` to disable OAuth authentication.
- On Ubuntu **B**, change IP address of **A** in the `~/.qkd/qnl/routes.json`.
```json
{
  "adjacent": {
    "A": "192.168.2.207"
  },
  "nonAdjacent": {
  }
}
```

## Running the network
- On Ubuntu A, `cd qkd-net/kms`, run command `./scripts/run`.
- On Ubuntu B, `cd qkd-net/kms`, run command `./scripts/run`

### Verifying qkd-net is running.
You can check if OpenQKDNetwork is up by 
- Looking at  `~/.qkd/mapping.log`
```txt title="~/.qkd/mapping.log"
{"192.168.2.207":"A","192.168.2.212":"B"}
```
- Looking at `~/qkd_logs/lsrp.log`
```txt title="~/qkd_logs/lsrp.log"
========Nodes/Links========
A
    A <----> B = 1
B
    B <----> A = 1
========Nodes/Links========
```

:::danger
If you cannot see these 2 files, then please stop & don't proceeed further. [Troubleshoot](#troubleshooting) if need be, but please first get this working.
:::


## Running a demo application
Now we will run **tls-demo application** (which can be found in `applications/tls-kms-demo`). This involves running two exectubles: `alice` on Ubuntu A and `bob` on Ubuntu B
- On both Ubuntu A & B, `cd ../applications/tls-demo/`, run `make` command
- On Ubuntu B, run command `./bob -b 10446`
- On Ubuntu A, run command `./alice -i 192.168.2.212 -b 10446 -f ./data/whale.mp3`

On Ubuntu A, you should see something like below

```bash
 -- Successfully conected to Bob
    -- Received PSK identity hint 'B 5048deac-4aa0-42a3-9b9a-f9d01b8a884d 0'
post String siteid=B&index=0&blockid=5048deac-4aa0-42a3-9b9a-f9d01b8a884d
HTTP-FETCH-REQUEST, url:http://localhost:8095/api/getkey,post:siteid=B&index=0&blockid=5048deac-4aa0-42a3-9b9a-f9d01b8a884d

HTTP-FETCH-RESPONSE:{"index":0,"hexKey":"36b4bf06cc5cbb7e61983f8b4dfd7771edc2ee01bb049b52e0a91a4f2e90fa0c","blockId":"5048deac-4aa0-42a3-9b9a-f9d01b8a884d"}

index: 0
key: 36b4bf06cc5cbb7e61983f8b4dfd7771edc2ee01bb049b52e0a91a4f2e90fa0c
blockId: 5048deac-4aa0-42a3-9b9a-f9d01b8a884d
    -- SHA1 of the received key : F7DC264E8C6165228EF9E0EBF0EFCF50B217B4D3
```

On Ubuntu B, file is received and saved in **bobdemo** and should see something like below
```bash
 -- Bob is listening for incomming connections on port 10446 ...
key_post : siteid=A
HTTP-FETCH-REQUEST, url:http://localhost:8095/api/newkey,post:siteid=A

HTTP-FETCH-RESPONSE:{"index":0,"hexKey":"36b4bf06cc5cbb7e61983f8b4dfd7771edc2ee01bb049b52e0a91a4f2e90fa0c","blockId":"5048deac-4aa0-42a3-9b9a-f9d01b8a884d"}

index: 0
key: 36b4bf06cc5cbb7e61983f8b4dfd7771edc2ee01bb049b52e0a91a4f2e90fa0c
blockId: 5048deac-4aa0-42a3-9b9a-f9d01b8a884d
SSL-Server-PSK-Hint:B 5048deac-4aa0-42a3-9b9a-f9d01b8a884d 0
    -- SHA1 of the received key : F7DC264E8C6165228EF9E0EBF0EFCF50B217B4D3
 -- TLS Closed
 -- Connection Closed
 -- Accept Connection
```

### Extending the setup to 3 nodes
- For 3 nodes system, extract `qkd-kaiduan-c.tar` on Ubuntu, assume we will run alice on A and bob on C, the network topology looks below,
> `A  <----> B <---> C`

Assume IP address of **C** is **192.168.2.235**, Add C to **B**'s adjacent nodes
```json
{
  "adjacent": {
    "A": "192.168.2.207",
    "C": "192.168.2.235"
  },
  "nonAdjacent": {
  }
}
```
## Testing KMS via Curl
We don't have to deal with OAuth at all in this branch :-D.
There are two REST API endpoints:

### 1. New Key

Application at site B makes a *newkey* call.

**Request**
```
  Method :      POST
  URL path :    /api/newkey
  URL params:   siteid=[alhpanumeric] e.g. siteid=A
```

**Response**
Format in JSON
```json
{
  index: Index of he key
  hexKey: Key in hexadecimal format
  blockId: Id of the block containing the key
}
```

Example request-response:

```shell
curl 'http://localhost:8095/api/newkey?siteid=A' | jq
```

```json
{
  "index": 28,
  "hexKey": "a2e1ff3429ff841f5d469893b9c28cbcb586d55c8ecf98c83c704824e889fc43"
  "blockId": "133ad3f8de6cc9da"
}
```

### 2. Get Key

Application on site A makes the getkey* call by using part of the response from the
newkey call above.

NOTE:
Since the sites are different hence OAuth tokens will be different, which
means a new OAuth token like above is fetched first by the application on site A.

**Request**
```text
  Method :      POST
  URL path :    /api/getkey
  URL params:   siteid=[alhpanumeric] e.g. siteid=B
                blockid=
                index=[Integer]
```

***Response***
Format in JSON
```text
  {
    index: Index of he key
    hexKey: Key in hexadecimal format
    blockId: Id of the block containing the key
  }
```

e.g. request-response:

```shell
curl 'http://localhost:8095/api/getkey?siteid=B&index=1&blockid=' | jq
```

```json
{
  "index": 1,
  "hexKey": "4544a432fb045f4940e1e2fe005470e1a35d85ede55f78927ca80f46a0a4b045"
  "blocId": "8a94dc22e761de8c5501addc05"
}
```

## Configuration files
Please read [this part](./config-files) of the docs

## Helper scripts
For building the services,

```shell
cd <top level director>/qkd-net/kms
./scripts/build
```

For running the services

```
cd <top level director>/qkd-net/kms
./scripts/run
```

### Checking registration service
Open a browser, check http://localhost:8761/

## Troubleshooting

### Stopping things
Stop OpenQKDNet by cd into `qkd-net/kms` and running `./scripts/stop` and `./scripts/cleanup`.

It is alright if on running`./scripts/stop` you get `No screen session found`.

### Ensure the nodes can talk to each other.
We'll use netcat to check whether the 2 nodes can connect to each other on TCP Port 9395.

- On Ubuntu B run `nc -l 9395`,
- On Ubuntu A run `nc IP_addr_of_node_B 9395`.

Enter any text on A. If you get the same text on B, then we know that the 2 nodes are able to talk to each other.

### Inspecting running Java processes
Ideally, once you've run './scripts/run`, the number of Java processes on both nodes should be `13`.

You can get the number of Java processes running on each node by running `ps -aux | grep java | wc -l` on both nodes.

If you get a number which is more than this, run `ps -aux | grep java` and figure out which other processes are running. If the process has no side effects for OpenQKDNet (e.g. VS Code Plugin related Java process), then you're good.

But, in particular, if you can see any process related to **jitsi** and/or **jicofo** running, we want to stop them via:
- `sudo systemctl stop jicofo`
- `sudo systemctl stop jitsi-videobridge2`