# How to install HarshiCorp Vault

### Link reference
- https://devopscube.com/setup-hashicorp-vault-beginners-guide/
- https://www.vaultproject.io/docs/configuration/index.html
- https://www.vaultproject.io/downloads.html

## 1. Install with docker

- Create file config use backend MySQL to store

    ```json
    {  
    "listener":{
        "tcp":{
            "address":"0.0.0.0:8200",
            "tls_disable":1
        }
    },
    "api_addr":"https://0.0.0.0:8200",
    "cluster_addr":"https://0.0.0.0:8201",
    "storage":{  
        "mysql":{  
            "address":"mysql",
            "database":"vault",
            "username":"vault",
            "password":"vaultpassword"
        }
    },
    "ui":true
    }
    ```
- Create docker-compose.yml

    ```yaml
    version: '3'

    services:
      mysql:
        image: mariadb
        environment:
          MYSQL_ROOT_PASSWORD: rootpassword
          MYSQL_DATABASE: vault
          MYSQL_USER: vault
          MYSQL_PASSWORD: vaultpassword
        volumes:
          - $PWD/data:/var/lib/mysql
        container_name: mysql

      vault:
        image: vault:1.0.3
        cap_add:
          - IPC_LOCK
        environment:
          VAULT_REDIRECT_INTERFACE: "eth0"
          VAULT_CLUSTER_INTERFACE: "eth0"
        #VAULT_LOCAL_CONFIG: |
        #  {
        #    "storage":{"mysql":{"address":"mysql","database":"vault","username":"vault","password":"Sieunhangao@1"}},
        #    "listener":{"tcp":{"address":"0.0.0.0:8200","tls_disable":1}},
        #    "cluster_name":"vault-lab-appota"
        #  }

        volumes:
          - $PWD/config.json:/vault/config/config.json
        command:
          - server
        ports:
          - 8200:8200
          - 8201:8201
        depends_on:
          - "mysql"
        container_name: vault
    ```

- Start service

    ```bash
    #!/bin/bash
    # Start mysql first
    docker-compose up -d mysql

    # Wait for mysql start success
    sleep 20

    # Start vault server
    docker-compose up -d vault
    ```
- Output start Vault Success

    ```bash
    docker-compose logs vault
    Attaching to vault
    vault    | Using eth0 for VAULT_REDIRECT_ADDR: http://172.25.0.3:8200
    vault    | Using eth0 for VAULT_CLUSTER_ADDR: https://172.25.0.3:8201
    vault    | ==> Vault server configuration:
    vault    | 
    vault    |              Api Address: http://172.25.0.3:8200
    vault    |                      Cgo: disabled
    vault    |          Cluster Address: https://172.25.0.3:8201
    vault    |               Listener 1: tcp (addr: "0.0.0.0:8200", cluster address: "0.0.0.0:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
    vault    |                Log Level: info
    vault    |                    Mlock: supported: true, enabled: true
    vault    |                  Storage: mysql (HA disabled)
    vault    |                  Version: Vault v1.0.3
    vault    |              Version Sha: 85909e3373aa743c34a6a0ab59131f61fd9e8e43
    vault    | 
    vault    | ==> Vault server started! Log data will stream in below:
    vault    | 
    vault    | 2019-03-03T05:59:26.558Z [INFO]  core: seal configuration missing, not initialized
    vault    | 2019-03-03T05:59:26.558Z [INFO]  core: security barrier not initialized
    ```

## 2. Unseal to startup Vault

### a. Unseal via CLI

- ***Install vault CLI***
    ```bash
    #!/bin/bash

    # Install vault client
    wget https://releases.hashicorp.com/vault/1.0.3/vault_1.0.3_linux_amd64.zip
    unzip vault_0.10.3_linux_amd64.zip -d .
    mv vault /bin
    
    # Install vault completion
    vault -autocomplete-install

    # Export URL to vault API
    export VAULT_ADDR=http://192.168.10.50:8200

    # Check vault status
    vault status
    ```

    - Output:
    ```bash
    Key                Value
    ---                -----
    Seal Type          shamir
    Initialized        false
    Sealed             true
    Total Shares       0
    Threshold          0
    Unseal Progress    0/0
    Unseal Nonce       n/a
    Version            n/a
    HA Enabled         false
    ```

- ***Init Operator vault***
    - Defautl it will generate `5 Unseal key` and only `1 Root Token`
    - Use will use `3 of 5 unseal key` to start vault server

    ```bash
    vault operator init > initfile
    vault status
    ```
    - Output
    ```
    Key                Value
    ---                -----
    Seal Type          shamir
    Initialized        true
    Sealed             true
    Total Shares       5
    Threshold          3
    Unseal Progress    0/3
    Unseal Nonce       n/a
    Version            1.0.3
    HA Enabled         false
    ```
    - Content of `initfile`
    ```
    Unseal Key 1: dIYQxEI8OF1Hlr5q2Jojepl85w3hFEC/nwoa2IyXNtjK
    Unseal Key 2: a/1u5yx9FL5Lp05X+XImKHeoOsgjbcNjdYWb8ey39oKu
    Unseal Key 3: LL713OODIJcXdPCbBbkfk8+hgAk/3TQvmQewTr1XtnX/
    Unseal Key 4: DO/qxspgbhamcpezLvstZjF4WdS6rKxSzCfGBj2sP5gB
    Unseal Key 5: ZdgkjZDHAFA6hytyTNdETjeWH24FYTZ9UyCbmfB5MCtr

    Initial Root Token: s.XSWcoIOz2IZDBXns5jE4TJ7X

    Vault initialized with 5 key shares and a key threshold of 3. Please securely
    distribute the key shares printed above. When the Vault is re-sealed,
    restarted, or stopped, you must supply at least 3 of these keys to unseal it
    before it can start servicing requests.

    Vault does not store the generated master key. Without at least 3 key to
    reconstruct the master key, Vault will remain permanently sealed!

    It is possible to generate new unseal keys, provided you have a quorum of
    existing unseal keys shares. See "vault operator rekey" for more information.
    ```

- ***Unseal vault server***

    ```bash
    # Unseal with 3 of 5 key above
    vault operator unseal dIYQxEI8OF1Hlr5q2Jojepl85w3hFEC/nwoa2IyXNtjK
    vault operator unseal a/1u5yx9FL5Lp05X+XImKHeoOsgjbcNjdYWb8ey39oKu
    vault operator unseal LL713OODIJcXdPCbBbkfk8+hgAk/3TQvmQewTr1XtnX/

    # Check status 
    vault status
    ```
    - Output

    ```bash
    # First key
    vault operator unseal dIYQxEI8OF1Hlr5q2Jojepl85w3hFEC/nwoa2IyXNtjK
    ---                -----
    Seal Type          shamir
    Initialized        true
    Sealed             true
    Total Shares       5
    Threshold          3
    Unseal Progress    1/3
    Unseal Nonce       e27ac261-96eb-2598-7109-7b15d38091f6
    Version            1.0.3
    HA Enabled         false

    # Second key
    vault operator unseal a/1u5yx9FL5Lp05X+XImKHeoOsgjbcNjdYWb8ey39oKu
    Key                Value
    ---                -----
    Seal Type          shamir
    Initialized        true
    Sealed             true
    Total Shares       5
    Threshold          3
    Unseal Progress    2/3
    Unseal Nonce       e27ac261-96eb-2598-7109-7b15d38091f6
    Version            1.0.3
    HA Enabled         false
    
    # Third key
    vault operator unseal LL713OODIJcXdPCbBbkfk8+hgAk/3TQvmQewTr1XtnX/
    Key             Value
    ---             -----
    Seal Type       shamir
    Initialized     true
    Sealed          false
    Total Shares    5
    Threshold       3
    Version         1.0.3
    Cluster Name    vault-cluster-081dfc54
    Cluster ID      db7c90c9-82ab-27b2-4d62-2e6436bfd402
    HA Enabled      false

    # Check status again
    vault status
    Key             Value
    ---             -----
    Seal Type       shamir
    Initialized     true
    Sealed          false
    Total Shares    5
    Threshold       3
    Version         1.0.3
    Cluster Name    vault-cluster-081dfc54
    Cluster ID      db7c90c9-82ab-27b2-4d62-2e6436bfd402
    HA Enabled      false
    ```

### b. Unseal via WEB UI

- After install , you can access address: http://192.168.10.50:8200/ui and staret unseal

    ![Unseal Vault](images/unseal.png)

- ***`Notes: Download init key to local and save carefully`***

## 3. First Login

- Use Root Token to Login

    ![First Login](images/first_login.png)