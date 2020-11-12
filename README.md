# Network automation using GitHub Actions


## Project Overview
Mass configuration push tool for networking devices using GitHub Actions self-hosted runners and the Netmiko Python library.


## Components
- GitHub Actions
- GHA self-hosted runners
- Ubuntu 20.0.4
- Docker engine
- Opengear access server
- Netmiko Python library


## Usage
**Diagram**
![](/diagram-network-automation-github-actions.png)

**Workflow using GitHub Desktop**  
- clone this repo
- create a new branch and publish it
- modify the device config file(s)
- commit changes to the new branch and push to origin
- create a pull request
- pull request review/approval
- merge pull request to update the Main branch
- delete the new branch
- GitHub Actions workflow starts
- self-hosted runner runs the jobs
- Python scripts are executed
- networking devices are updated


## Install on Ubuntu 20 Docker container
**Download and install Ubuntu**
```
docker pull ubuntu
docker run -it ubuntu
```
**Update and install packages**
```
apt update
apt upgrade

apt install apt-utils
apt install curl 
apt install iputils-ping  
apt install net-tools
apt install python3-pip
apt install software-properties-common
apt install sudo
apt install vim

pip3 install --upgrade pip
pip3 install netmiko
```

**[Install GitHub CLI](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)**
```
apt-key adv --keyserver keyserver.ubuntu.com --recv-key C99B11DEB97541F0
apt-add-repository https://cli.github.com/packages
apt update
apt install gh
```
*Create a Personal Access Token: GitHub > profile pic > Settings > Developer settings > Personal access tokens > Generate new token*  
```
gh auth login (GitHub.com > Paste an authentication token > PASTE > HTTPS)
```

*Cache GitHub credentials in Git*  
```
git config --global credential.helper cache
git config --global credential.helper 'cache --timeout=28800'
```

**Install and run a GHA self-hosted agent**
```
adduser siteadmin
usermod -aG sudo siteadmin
su siteadmin

sudo mkdir actions-runner
sudo chown siteadmin:siteadmin actions-runner
cd actions-runner
```
```
curl -O -L https://github.com/actions/runner/releases/download/v<RUNNER_VERSION>/actions-runner-linux-x64-<RUNNER_VERSION>.tar.gz
tar xzf ./actions-runner-linux-x64-<RUNNER_VERSION>.tar.gz
```
*Get the token from: GitHub > repo > Settings > Actions > Add runner*
```
./config.sh --url https://github.com/Davitiani/network-automation-github-actions --token <TOKEN>
./run.sh
```

**Uninstall**  
`./config.sh remove --token <TOKEN>`


## Issues
- [ ] standardize on deleting the repo folder vs keeping it and doing a git pull  
- [ ] git credentials cache timeout max value
