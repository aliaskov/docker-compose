# Ci-Server @ AWS-EC2 #

## Setup From Scratch ##

- create an ec2-instance
- create 4 volumns:
  1. name: Jenkins, size: 64 GB, type: gp2
  2. name: jenkins_jenkins-master, size: 8 GB, type: gp2
  3. name: jenkins_jenkins-slave, size: 160 GB, type: gp2
  4. name: artifactory_fd-artifactory, size: 8 GB, type: standard
- attach them to the ec2-instance:
  1. /dev/sda1
  2. /dev/sdf
  3. /dev/sdg
  4. /dev/sdh
- create the directories for the mount points:
  2. /opt/jenkinsMaster
  3. /opt/jenkinsSlave
  4. /opt/artifactoryStorage
- put the mounts into the /etc/fstab:
  ```sh
  UUID=711e1ec2-2a36-4405-bf46-44b43cfee42e / ext4 defaults 1 1
  /dev/xvdf /opt/jenkinsMaster ext4 defaults,nofail 0 2
  /dev/xvdg /opt/jenkinsSlave ext4 defaults,nofail 0 2
  /dev/xvdh /opt/artifactoryStorage ext4 defaults,nofail 0 2
  ```
- execute:
  ```sh
  sudo mount -a
  ```
- set up docker-compose
  - create a directory /opt/ci-server
  - run docker-compose.yml 
  - execute:
    ```sh
    touch /opt/traefik/acme.json && chmod 600 /opt/traefik/acme.json
    ```
  - pull traefik.toml
- create the a records in Route53 for
  - "build",
  - "artifactory" and
  - "vault"
- for retrieving the certificates via traefik from letsencrypt, port 80 have to be open for the hole world, e. g. 0.0.0.0/0
- finally execute:
  ```sh
  docker-compose up -d
  ```
- adding special user for artifactory on ec2-instance (docker host):
  ```sh
  useradd -u 1030 artifactory
  ```

## Vault ##
The Vault needs some extras. By default vault data is sealed after restart

These steps have to be done for unsealing:
1) start containers vauilt-app and vault-web
2) ssh into container  
```sh
docker exec -it vault-app ash
```
3) vault status
```
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
4) execute:
```
vault init
```
Unseal Key 1: (look for it in confluence)
Unseal Key 2: (look for it in confluence)
Unseal Key 3: (look for it in confluence)
Unseal Key 4: (look for it in confluence)
Unseal Key 5: (look for it in confluence)

Initial Root Token:(look for it in confluence)

5) execute 3(!) times:
```sh
vault operator unseal  
```
And enter Unseal Key 1,Unseal Key 2 and Unseal Key 3
Than you will see in status -
```Sealed  false
```
6) execute:
```
vault login

Token (will be hidden):
Success! You are now authenticated. The token information displayed below is already stored in the token helper. You do NOT need to run "vault login" again. Future Vault requests will automatically use this token.
```
7) via vault ui - changed auth type to token. use Initial Root Token:(look for it in confluence) to log in as root

## vault login with username and password ##
To enable username and password auth: 
```sh 
vault auth enable userpass
Success! Enabled userpass auth method at: userpass/
```
add user with login and password with default policy (permissions)

```sh
vault write auth/userpass/users/USERNAME policies=default password=
```
add admin with login and password with admin policy (permissions)
```sh
vault write auth/userpass/users/user policies=admin password=*********
Success! Data written to: auth/userpass/users/user
```

## vault login with github token ##
To enable github login auth:

```sh vault auth enable github
Success! Enabled github auth method at: github/
```
give access to organization
```sh 
vault write auth/github/config organization=********
```
admin policy for github user 
```sh
vault write auth/github/map/users/user value=admin
```

every member of  team can log in using github token and receive team policy (create,read,list)
```sh
vault write auth/github/map/teams/team value=team
```

## how to get github token ##

In the upper-right corner of any page, click your profile photo, then click Settings.
In the left sidebar, click Developer settings.
In the left sidebar, click Personal access tokens.
Click Generate new token.
Give your token a descriptive name.
Select the scopes, or permissions, you'd like to grant this token. To use your token to access repositories from the command line, select repo.
Click Generate token.
Click  to copy the token to your clipboard. 
