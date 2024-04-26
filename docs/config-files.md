---
sidebar_label: 'Configuration files'
sidebar_position: 3
---
# Understanding the Configuration files
As this is a multi-layer distributed system, a few configuration files are used to facilitate the deployment for different nodes (or sites). They need to be carefully checked before running/testing the system.

### kms.conf
Sample applications provided (qTox and tls-kms-demo) require kms.conf to be present under
the $HOME/.qkd directory.

This file can be manually created by puting following entries:
```text title=kms.conf
http://localhost:9992/uaa/oauth/token
http://localhost:8095/api/newkey
http://localhost:8095/api/getkey
C
false
```
We expect this (`~/.qkd/kms.conf`) to be identical to `.qkd/kms/kms.conf`.

Depending on where the KMS node is deployed, `localhost` can be replaced by the IP address of
the host running the KMS layer.
The last line represents the KMS site id, *which should be updated with the
correct site id*. Above is just an example.

### config-repo

config-repo directory under `$HOME/` contains all the configuration property
files required by all the microservices.

All the files reside under `<top level directory>/qkd-net/kms/config-repo` from where
they are copied to `$HOME/config-repo` and checked in a local git repository.

### site.properties

This file is located under `<top level directory>/qkd-net/kms/kms-service/src/main/resources/`.

Explanation of site specific configuration properties used by KMS service:
```text title="site.properties"
// Number of keys per block
kms.keys.blocksize=1024

// Size of a key in bytes
kms.keys.bytesize=32

// Top level location for locating key blocks
kms.keys.dir=${HOME}/.qkd/kms/pools/

// IP address of the host where key routing service is running
qnl.ip=localhost

// Port on which key routing service is listening for key block requests
qnl.port=9292
```

### KMS QNL Service's config.yaml

This file is copied from `<top level directory>/qkd-net/kms/kms-qnl-service/src/main/resources/` to `$HOME/.qkd/kms/qnl` from where it is used by KMS QNL service


Expalantion of configuration properties used by KMS QNL service:

```yaml title="~/.qkd/kms/qnl/config.yaml"
# Port on which KMS QNL service is listening
port: 9393

# Size of a key in bytes
keyByteSz: 32

# Number of keys in a block. Key routing service provides KMS keys in blocks of size    
keyBlockSz: 1024

# Location where KMS QNL service puts the pushed key blocks from key-routing service
poolLoc: kms/pools
```
### config.yaml and routes.json (Key-Routing Service)

### routes.json

This file is copied from `<top level directory>/qkd-net/qnl/conf/` to
`$HOME/.qkd/qnl`.

Topology information is contained in `route.json`.
This information is populated manually for adjacent (neighboring) nodes, while the non-adjacent nodes will be discovered by the routing module. Adjacent nodes section contains the name of the node as key and it's IP address as the value.

Example `route.json` file for a QNL Node "B":
```json title="~/.qkd/qnl/routes.json"
{
  "adjacent": {
    "A": "192.168.1.101",
    "C": "192.168.1.103"
  },
  "nonAdjacent": {
  }
}
```

### config.yaml

This file is copied from `<top level directory>/qkd-net/qnl/conf/` to
`$HOME/.qkd/qnl`.  It contains various configuration parameters for the key routing service
running as a network layer.

Brief explanation of various properties is given with the following example file:

```yaml title="~/.qkd/qnl/config.yaml"
# Base location for finding other paths and configuration files.
base: .qkd/qnl

# Route configuration file name.
routeConfigLoc: routes.json

# Location where QLL puts the key blocks for QNL to carve out key blocks for KMS.
qnlSiteKeyLoc: qll/keys

# Each site has an identifier uniquely identifying that site.
siteId: A

# Port on which key routing service is listening for key block requests.
port: 9292

# Size of a key in bytes.
keyBytesSz: 32

# Number of keys in a block. Key routing service provides KMS keys in blocks of size.  
keyBlockSz: 1024

# Key routing service expects QLL to provide keys in blocks of qllBlockSz.
qllBlockSz: 4096

# IP address of the host running KMS QNL service.
kmsIP: localhost

# Port on which KMS QNL service is listening.
kmsPort: 9393

# One Time Key configuration.
OTPConfig:
    # Same as above
    keyBlockSz: 1024
    
    # Location of the OTP key block.
    keyLoc: otp/keys
```