# Run a Validator Node in the SKALE Network

This page is the step by step guide that shows how to run a validator node in the SKALE Network

<StepsLayout id='Validator'>

<StepsController>
    <StepNav stepId='one' label='Register\nValidator'><SendTransaction/></StepNav>
    <StepNav stepId='two' label='Setup\nSGX Wallet'><ThresholdSignatures/></StepNav>
    <StepNav stepId='three' label='Setup\nSKALE Node'><LeaderlessConsensus/></StepNav>
    <StepNav stepId='four' label='Register\nNode in SKALE Network'><Request/></StepNav>
    <StepNav stepId='five' label='Upload new SSL Certificates'><ByzantineFaultTolerant/></StepNav>
</StepsController>

<Step id='one'>

## 1. Register Validator with SKALE Validator CLI

SKALE Validator CLI is the validator client interface for registering a new validator into network or handling additional delegation services where validators can self delegate or token holders can delegate to a validator. These are the type of operations that can be done with the Validator CLI:

-   Register Validator (Set Commission Rate or Minimum delegation amount)
-   Accept pending delegations
-   Link all validator node addresses to a validator wallet address
-   Request or cancel a delegation

See the SKALE Validator CLI code and documentation on [**GitHub**](https://github.com/skalenetwork/validator-cli).

This document contains instructions on how to get started with the SKALE Validator CLI.

PS: Validator CLI doesn't have to be installed in the same server as the node-cli!

### Step 1.1: Install SKALE Validator CLI

#### Download the SKALE Validator CLI binary

VERSION_NUM is a version identifier

**Terminal Command:**

```shell
VERSION_NUM=[VERSION_NUM] && sudo -E bash -c "curl -L https://github.com/skalenetwork/validator-cli/releases/download/$VERSION_NUM/sk-val-$VERSION_NUM-`uname -s`-`uname -m` >  /usr/local/bin/sk-val"
```

#### Apply executable permissions to the binary

**Terminal Command:**

```shell
chmod +x /usr/local/bin/sk-val
```

#### Get SKALE Manager contracts info and set the endpoint

**Terminal Command:**

```shell
sk-val init -e [ENDPOINT] -c [ABI] --wallet [software/ledger]
```

Required arguments:

-   `--endpoint/-e` - RPC endpoint of the node in the network where SKALE manager is deployed (`http` or `https`)
                    Example: <https://my.geth.node.ip/..>.

-   `--contracts-url/-c` - URL to SKALE Manager contracts ABI and addresses

-   `-w/--wallet` - Type of the wallet that will be used for signing transactions (software or ledger)

### Step 1.2: Setup wallet

#### Software wallet

If you want to use software wallet you need to save private key into a file.

Replace `[YOUR PRIVATE KEY]` with your wallet private key

**Terminal Command:**

```shell
echo [YOUR PRIVATE KEY] > ./pk.txt
```

#### Ledger  wallet

If you want to use ledger you should install ETH ledger application and  initilize device with `setup-ledger` command.

**Terminal Command:**

```shell
sk-val wallet setup-ledger --address-index [ADDRESS_INDEX] --keys-type [KEYS_TYPE]
```

Required arguments:

-   `--address-index` - Index of the address to use (starting from `0`)
-   `--keys-type` - Type of the Ledger keys (live or legacy)

> Make sure you enabled contracts data sending on ETH application. Otherwise transactions won't work

### Step 1.3: Register as a new SKALE validator

> DON'T REGISTER A NEW VALIDATOR IF YOU ALREADY HAVE ONE! check : `sk-val validator ls`. For additional node set up, please go to Step 3.5.

**Terminal Command:**

```shell
sk-val validator register -n [NAME] -d [DESCRIPTION] -c [COMMISSION_RATE] --min-delegation [MIN_DELEGATION] --pk-file ./pk.txt
```

Required arguments:

-   `--name/-n` - Validator name
-   `--description/-d` - Validator description (preferably organization info)
-   `--commission-rate/-c` - Commission rate (percent %)
-   `--min-delegation` - Validator minimum delegation amount

Optional arguments:

-   `--pk-file` - Path to file with private key (only for `software` wallet type)
-   `--gas-price` - Gas price value in Gwei for transaction (if not specified doubled average network value will be used)
-   `--yes` - Confirmation flag

### Step 1.4: Make sure that the validator is added to the whitelist

Note: This is for testing purposes only. [**Whitelist**](https://alpine.skale.network/whitelist)

### Step 1.5: Write down your Node Address (SGX Wallet Address)

After executing following command you will find Node Address (Sgx Wallet Address)

**Terminal Command:**

```shell
 skale wallet info
```

**Output:**

```shell
root@se-test-01:~# skale wallet info
--------------------------------------------------
Address: 0x.... -> ThisIsYour_NodeAddress
ETH balance: 3.499059343 ETH
SKALE balance: 200 SKALE
--------------------------------------------------
```

Please copy your SGX Wallet Address, you will be using it for linking node address to validator address.

</Step>

<Step id='two'>

## 2. Set up SGX Wallet

Sgxwallet runs as a network server. Clients connect to the server, authenticate to it using TLS 1.0 protocol with client certificates, and then issue requests to the server to generate crypto keys and perform cryptographic operations. The keys are generated inside the secure SGX enclave and never leave the enclave unencrypted.

To be able to set up an SGX Wallet, validators are required to have an SGX compatible servers. Before installing SGX Wallet, validators has to make sure that SGX is enabled in the server.

SKALE will have two types of SGX operations:

-   **Local (Secure)**: Wallet running on the same server as sub-node.
-   **Network**: Sub-node talks to SGX wallet over the SKALE Network. The validator is responsible for securing the connection. If validator is planning to have a separate SGX compatible node than the Blockchain node, SGX Wallet node doesn't have to have the same hardware requirements as the sub-node. SGXWallet doesn't require a lot of computational power. After setting up the Network SGX node, enable SSL certification before adding the url to configuration in SKALE Node Set up.

## SKALE SGX Wallet

SGX Wallet setup is the first step of the Validator Node registration process.  ‍

Sgxwallet is a next generation hardware secure crypto wallet that's based on Intel SGX technology. It currently supports Ethereum and SKALE.

**SGX is a secure storage for BLS private key shares. It would be used inside consensus to sign new blocks. But SGX isn't only used for private key shares. For more information, please check** [**here.**](/validators/requirements)

**SKALE DKG uses Intel® SGX server to store account and BLS keys and all the data related to DKG process and it also uses the random number generator provided by Intel® SGX. For more information, please check** [**here.**](/technology/skale-dkg)

Sgxwallet runs as a network server. Clients connect to the server, authenticate to it using TLS 1.0 protocol with client certificates, and then issue requests to the server to generate crypto keys and perform cryptographic operations. The keys are generated inside the secure SGX enclave and never leave the enclave unencrypted.

The server provides an initial registration service to issue client certificates to the clients. The administrator manually approves each registration

### **Prerequisites**

-   Ubuntu 18.04 (Ubuntu > 18.04 not yet supported)
-   SGX-enabled Intel processor
-   At least 8GB RAM
-   Swap size equals to half of RAM size
-   Ports 1026–1029

**Terminal Commands:**

```shell
sudo apt-get install build-essential make cmake gcc g++ yasm  python libprotobuf10 flex bison automake libtool texinfo libgcrypt20-dev libgnutls28-dev
```

**Install Docker:**

```shell
sudo apt-get install -y docker
```

**Install docker.io:**

```shell
sudo apt-get install -y docker.io
```

**Install docker-compose:**

```shell
sudo apt-get install -y docker-compose
```

**Install cpuid and libelf-dev packages:**

```shell
sudo apt-get install -y libelf-dev cpuid
```

**Verify your processor supports Intel SGX with:**

```shell
cpuid | grep SGX:
```

**Output**

```shell
SGX: Software Guard Extensions supported = true
```

**Important note:**  
After installing docker make sure that `live-restore` option
is enabled in `/etc/docker/daemon.json`. See more info in the [docker docs](https://docs.docker.com/config/containers/live-restore/).  

* * *

### Set Up SGX Wallet

#### STEP 1 - Clone SGX Wallet Repository

Clone SGX Wallet Repository to your SGX compatible Server:

```shell
git clone https://github.com/skalenetwork/sgxwallet/
cd sgxwallet
git checkout tags/1.58.5-stable.1
```

#### STEP 2 - Enable SGX

**SGX Wallet repository includes the sgx_enable utility. To enable SGX run:**

```shell
sudo ./sgx_enable
```

Note: if you aren't using Ubuntu 18.04 (not recommended), you may need to rebuild the sgx-software-enable utility before use by typing:

```shell
cd sgx-software-enable;
make
cd ..
```

**Install SGX Library:**

```shell
cd scripts
sudo ./sgx_linux_x64_driver_2.5.0_2605efa.bin
cd ..
```

**System Reboot:**

> Reboot your machine after driver install!

**Check driver installation:**
To check that isgx device is properly installed run this command:

```shell
ls /dev/isgx
```

If you don't see the isgx device, you need to troubleshoot your driver installation from [**here.**](https://github.com/skalenetwork/sgxwallet/blob/develop/docs/enabling-sgx.md)

**Another way to verify Intel SGX is enabled in BIOS:**

> **_If you already executed the previous steps please move to STEP 3_**

Enter BIOS by pressing the BIOS key during boot. The BIOS key varies by manufacturer and could be F10, F2, F12, F1, DEL, or ESC.

Usually Intel SGX is disabled by default.

To enable:

find the Intel SGX feature in BIOS Menu (it's usually under the "Advanced" or "Security" menu)
Set SGX in BIOS as enabled (preferably) or software-controlled.
save your BIOS settings and exit BIOS.
Enable "software-controlled" SGX
Software-controlled means that SGX needs to be enabled by running a utility.

#### STEP 3 - Start SGX

Production Mode

```shell
cd sgxwallet/run_sgx;
```

On some machines, the SGX device isn't **/dev/mei0** but a different device, such as **/dev/bs0** or **/dev/sg0**. In this case please edit docker-compose.yml on your machine to specify the correct device to use:

```shell
vi docker-compose.yml
```

make sure `image` is skalenetwork/sgxwallet:&lt;`SGX_VERSION`> in docker-compose and it will look like:

```shell
version: '3'
services:
  sgxwallet:
    image: skalenetwork/sgxwallet:<SGX_VERSION>
    ports:
      - "1026:1026"
      - "1027:1027"
      - "1028:1028"
      - "1029:1029"
    devices:
      - "/dev/isgx"
      - "/dev/sg0"
    volumes:
      - ./sgx_data:/usr/src/sdk/sgx_data
      -  /dev/urandom:/dev/random
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "4"
    restart: unless-stopped
    command: -s -y -V
    healthcheck:
      test: ["CMD", "ls", "/dev/isgx", "/dev/"]
```

**Start SGX Wallet Containers**
To run the server as a daemon:

```shell
sudo docker-compose up -d
```

The backup key is automatically stored in sgx_data directory of SGX Wallet.

The filename of the key is sgx_wallet_backup_key.txt, and is generated the first time the SGX wallet is started.

**This key must be securely recorded and stored.**
Be sure to store this key in a safe place, then go into a docker container and securely remove it with the following command:

```shell
docker exec -it <SGX_CONTAINER_NAME> bash && apt-get install secure-delete && srm -vz backup_key.txt
```

### STOP SGX Wallet Containers (for back up purposes)

```shell
cd sgxwallet/run_sgx
sudo docker-compose stop
```

> If you set up SGX wallet in a separate server than your SKALE Node, you should enable SSL/TLS for your SGX node. Make sure you finalize this before you move on to your next step.

</Step>

<Step id='three'>

## 3. Setup SKALE Node with SKALE Node CLI

After Setting up SGX Wallet and create certifications, validators can download the SKALE Node CLI executables register and maintain your SKALE node. This process downloads docker container images from docker hub and spins up SKALE Node functionalities. Some base containers such as SKALE Admin, Bounty, SLA, TransactionManager will be created during installation for each node.

See the SKALE Node CLI code and documentation on [**GitHub**](https://github.com/skalenetwork/skale-node-cli)\*‍

This document contains instructions on how to get started with the SKALE Node CLI.

### **Prerequisites**

-   A Linux x86_64 machine
-   SGX-enabled Intel processor
-   Ports 22, 8080, 9100, and 10000–11500, and ICMP IPv4 open for all
-   SSL Certificates
-   Ubuntu 18.04 or later LTS
-   2TB attached and unmounted storage (200GB Testnet)
-   8 core
-   32GB RAM
-   16GB swap
-   Install docker.io
-   Install docker-compose 
-   Install iptables-persistent - (for reinitializing base firewall rules after node machine was rebooted)
-   Make sure lvm2 package is installed (`dpkg -l | grep lvm2`)

**Important notes:**  

1.  After docker installation make sure that the `live-restore` option
    is enabled in `/etc/docker/daemon.json`. See more info in the [docker docs](https://docs.docker.com/config/containers/live-restore/).  
2.  If you have any issues you can save the logs using `skale logs dump` command.  
    It's also useful to check logs from node-cli `skale cli logs` from docker plugin `/var/log/docker-lvmpy/lvmpy.log` if there are any issues.
3.  You can install iptables-persistent using the following commands:

```shell
echo iptables-persistent iptables-persistent/autosave_v4 boolean true | sudo debconf-set-selections
echo iptables-persistent iptables-persistent/autosave_v6 boolean true | sudo debconf-set-selections
sudo apt install iptables-persistent -y
```

4.  You will need SSL certificates for each node. [See the Node SSL instructions](/validators/node-ssl.adoc).
5.  You should run skale commands using sudo.

If you have any concerns or questions, please don't hesitate to reach out to SKALE Team leads on [discord](http://skale.chat/).

[![Discord](https://img.shields.io/discord/534485763354787851.svg)](https://discord.gg/vvUtWJB)

### Step 3.1: Install SKALE Node CLI

#### Download the SKALE Node CLI binary

VERSION_NUM is a version identifier

**Terminal Command:**

```shell
VERSION_NUM=[VERSION_NUM] && sudo -E bash -c "curl -L https://github.com/skalenetwork/skale-node-cli/releases/download/$VERSION_NUM/skale-$VERSION_NUM-`uname -s`-`uname -m` >  /usr/local/bin/skale"

```

#### Make the SKALE Node CLI binary executable

**Terminal Command:**

```shell
sudo chmod +x /usr/local/bin/skale

```

### Step 3.2: Setup SKALE Node

#### Initialize SKALE node daemon and install dependencies

:warning: **Please avoid re-initialization**: First run `skale node info` to confirm current state of intialization..

Required options for the `skale node init` command:

-   `--install-deps` - install additional dependencies 

Required options for the `skale node init` command in environment file:

-   `SGX_SERVER_URL` - URL to SGX server in the network, can be used for current node if the current node supports intel technology SGX. SGX node can be set up through SGXwallet repository
-   `DISK_MOUNTPOINT` - Block device to be used for storing sChains data
-   `IMA_CONTRACTS_ABI_URL` - URL to IMA contracts ABI and addresses
-   `MANAGER_CONTRACTS_ABI_URL` - URL to SKALE Manager contracts ABI and addresses
-   `FILEBEAT_HOST` - URL to the Filebeat log server
-   `CONTAINERS_CONFIG_STREAM` - git branch with containers versions config
-   `DOCKER_LVMPY_STREAM` - git branch of docker lvmpy volume dirver for schains
-   `DB_PORT` - Port for of node internal database (default is 3306)
-   `DB_ROOT_PASSWORD` - root password
-   `DB_PASSWORD` - Password for root user of node internal database (equal to user password by default)
-   `DB_USER` - MySQL user for local node database
-   `IMA_ENDPOINT` - IMA endpoint to connect.
-   `ENDPOINT` - RPC endpoint of the node in the network where SKALE manager is deployed (`http` or `https`)

Create a `.env` file and specify following parameters:

**Terminal Command:**

```shell
    SGX_SERVER_URL=[SGX_SERVER_URL]
    DISK_MOUNTPOINT=[DISK_MOUNTPOINT]
    IMA_CONTRACTS_ABI_URL=[IMA_CONTRACTS_ABI_URL]
    MANAGER_CONTRACTS_ABI_URL=[MANAGER_CONTRACTS_ABI_URL]
    FILEBEAT_HOST=[FILEBEAT_HOST]
    CONTAINER_CONFIGS_STREAM=[CONTAINER_CONFIGS_STREAM]
    DOCKER_LVMPY_STREAM=[DOCKER_LVMPY_STREAM]
    DB_PORT=[DB_PORT]
    DB_ROOT_PASSWORD=[DB_ROOT_PASSWORD]
    DB_PASSWORD=[DB_PASSWORD]
    DB_USER=[DB_USER]
    IMA_ENDPOINT=[IMA_ENDPOINT]
    ENDPOINT=[ENDPOINT]
    NODE_CLI_SPACE=[NODE_CLI_SPACE]
    SKALE_NODE_CLI_VERSION=[SKALE_NODE_CLI_VERSION]
```

Please feel free to set values own  **DB_PASSWORD**, **DB_ROOT_PASSWORD**, **DB_USER**.

```shell
    TG_API_KEY - Telegram API key
    TG_CHAT_ID - Telegram chat ID
    MONITORING_CONTAINERS - True/False will enable monitoring containers (filebeat, cadvisor, prometheus)
                            Required for TestNets
```

**Terminal Command:**

```shell
skale node init .env --install-deps
```

**Output:**

```shell
48914619bcd3: Pull complete
db7a07cce60c: Pull complete
d285532a5ada: Pull complete
8646278c4014: Pull complete
3a12d6e582e7: Pull complete
0a3d98d81a07: Pull complete
43b3a182ba00: Pull complete
Creating monitor_cadvisor          ... done
Creating monitor_node_exporter     ... done
Creating monitor_filebeat          ... done
Creating skale_transaction-manager ... done
Creating config_base_1             ... done
Creating skale_watchdog            ... done
Creating skale_mysql               ... done
Creating skale_sla                 ... done
Creating skale_admin               ... done
Creating skale_bounty              ... done
Creating skale_api                 ... done
```

#### Show your SKALE SGX wallet info

This command prints information related to your sgx wallet. Node operates only from the sgx wallet:

**Terminal Command:**

```shell
skale wallet info

```

**Output:**

```shell
Address: <your-skale-private-net-wallet-address>
ETH balance: 0 ETH
SKALE balance: 0 SKALE

```

#### Check if your node is connected to sgx

**Terminal Command:**

```shell
skale sgx info

```

**Output:**

```shell
SGX server status:
┌────────────────┬──────────────────────────┐
│ SGX server URL │ <sgx-url>
├────────────────┼──────────────────────────┤
│ Status         │ CONNECTED                │
└────────────────┴──────────────────────────┘

```

### Step 3.3: Get Test Tokens to your SGX and Validator wallets\*\*

Get Tokens from the  [**SKALE Faucet**](https://faucet.skale.network/validators)

If you’re unable to transfer funds please feel free to reach out to the team on  [discord](http://http:skale.chat/).

[Click here for Faucet](https://faucet.skale.network/validators)

Once tokens have been transferred, please check your wallet in the terminal.

**Terminal Command:**

```shell
skale wallet info

```

### Step 3.4: Sign validator id using sgx wallet

Execute this command and find your validator ID

**Terminal Command:**

```shell
sk-val validator ls
```

Get your SKALE node signature. This SIGNATURE will be used in Step 4.6 while linking node addresses to your validator.

**Terminal Command:**

```shell
skale node signature [VALIDATOR_ID]

```

**Output:**

```shell
Signature: <your-signature>
```

Get your node address.

**Terminal Command:** 

```shell
skale wallet info
```

**Output:** 

```shell
--------------------------------------------------
Address: 0x... <- your address
ETH balance: 0.1 ETH
SKALE balance: 0 SKALE
--------------------------------------------------
```

### Step 3.5: Link skale wallet address to your validator account using validator-cli

> You can find node address by executing `skale wallet info` command

**Terminal Command:**

```shell
 sk-val validator link-address [NODE_ADDRESS] [SIGNATURE] --yes --pk-file ./pk.txt
```

Optional arguments:

-   `--pk-file` - Path to file with private key (only for `software` wallet type)
-   `--gas-price` - Gas price value in Gwei for transaction (if not specified doubled average network value will be used)
-   `--yes` - Confirmation flag

### Step 3.6: Send-Accept Delegation using validator-cli

This step 3.6 and 3.7 is for testing delegation flow. If MSR isn't set in the testnet please move to Step 4.

> Make sure you  already have at least 100 SKL tokens in your validator wallet for TestNet MSR is 100SKL tokens.

**Terminal Command:**

```shell
 sk-val holder delegate --validator-id [Validator_ID] --amount 100 --delegation-period 3 --pk-file pk.txt --info "please accept delegation" --yes
```

Required arguments:

-   `--validator-id` - Validator id
-   `--amount` - Delegation amount info)
-   `--delegation-period` - Delegation period (only 2 month allowed for now)
-   `--info` - Delegation info

Optional arguments:

-   `--pk-file` - Path to file with private key (only for software wallet
-   `--gas-price` - Gas price value in Gwei for transaction (if not specified doubled average network value will be used) type)
-   `--yes` - Confirmation flag

List your delegations

**Terminal Command:**

```shell
    sk-val validator delegations [VALIDATOR_ID]
```

You will see your pending delegation please get the delegation number and accept delegation

**Terminal Command:**

```shell
    sk-val validator accept-delegation --delegation-id [DELEGATION-ID] --pk-file pk.txt
```

Required arguments:

-   `--delegation-id` - Delegation id to accept

Optional arguments:

-   `--pk-file` - Path to file with private key (only for software wallet type)
-   `--gas-price` - Gas price value in Gwei for transaction (if not specified doubled average network value will be used)
-   `--yes` - Confirmation flag

List your delegations make sure your accepted delegations are equal or more than 100SKL tokens

```shell
    sk-val validator delegations [VALIDATOR_ID]
```

### Step 3.7 : Delegations have to be in "DELEGATED" status

To be able to register a node in the network with the MSR requirement your delegations have to be in the DELEGATED status.
After the previous step delegation status will be seen as accepted.
"Delegated" status will be automatically updated 1st day of each month when the epoc starts.

> For Testnet Only, please ask Core team to skip time to update delegation status

</Step>

<Step id='four'>

## 4: Setup SSL Certificates

Here is an overview of the process to setup SSL certificates. For more details, please see [Node SSL docs](/validators/node-ssl.adoc).

### Step 4.1: Setup IP redirects

You will need to setup redirects from each node IP to the node domain.

### Step 4.2: Verify redirects

Verify that redirects work by sending a request to the watchdog API on 3009 port using your domain name.

### Step 4.3: Issue SSL certificates

You will need SSL certs issued by one of the Trusted CAs. Once you've decided on the certificate issuer you have several options - issue a separate certificate for each subdomain (node-0.awesome-validator.com, node-1.awesome-validator.com) or issue a single Wildcard SSL for all nodes (\*.awesome-validator.com). As a result, you should have 2 main files saved and copied to the respective nodes:

-   Certificate file (for example, fullchain.pem or cert.pem)
-   Private key file (for example, privkey.pem, pk.pem)

### Step 4.4: Upload certificates to the SKALE Node

Once you copied the certificate and private key file, all you have to do is to run the following command:

```shell
skale ssl upload -c $PATH_TO_CERT_FILE -k $PATH_TO_KEY_FILE
```

### SSL Status

Status of the SSL certificates on the node

**Terminal Command:**

```shell
skale ssl status
```

Admin API URL: \[GET] `/api/ssl/status`

</Step>

<Step id='five'>

## 5: Register Node in SKALE Network

### Step 5.1: Register Node with Node CLI

Note: Before proceeding, you will need to have at least  **0.2 Test ETH** for Testnet. Also amount of delegated skale tokens need to be more or equal to minimum staking amount. Otherwise you won't be able to register with the SKALE Testnet.

Note: Also, be sure the node has SSL certificates!

To register with the network, you will need to provide the following:

1.  Node name
2.  Machine public IP
3.  Port - beginning of the port range that will be used for skale schains (10000 recommended)
4.  Domain name

**Terminal Command:**

```shell
skale node register --name [NODE_NAME] --ip [NODE_IP] --port [PORT] --domain [DOMAIN_NAME]

```

Optional arguments:

-   `--gas-price` - Gas price value in Gwei for transaction (if not specified doubled average network value will be used)

**Output:**

> Node registered in SKALE manager. For more info run: skale node info

### Step 5.2: Check Node Status

You can check the status of your node, and ensure that it's properly registered with the SKALE Network.

**Terminal Command:**

```shell
skale node info
```

**Output:**

```shell
# Node info
Name: <Node name>
ID: <Node ID>
IP: <IP of Machine>
Public IP: <Public IP of Machine>
Port: <Node port>
Domain name: <Node domain name>
Status: Active
```

</Step>
</StepsLayout>
