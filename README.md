# Network Automation Using GitHub Actions


## Project Overview
Network automation framework based on the following **[GitOps](https://www.gitops.tech/)** principles:
- all device configurations are defined as `code` and stored in a [distributed version control system](https://en.wikipedia.org/wiki/Distributed_version_control) repository
- configuration files are in **raw format** and use a [declarative](https://en.wikipedia.org/wiki/Declarative_programming) language syntax to describe the **desired** system state
- **GitHub** is assumed to be the [Single Source of Truth](https://en.wikipedia.org/wiki/Single_source_of_truth) - everything related to the application and the environment is documented here
- all configuration changes are initiated via **Git** and are implemented programmatically via **GitHub Actions**
- manual changes by directly modifying device configurations are **not permitted**
- configurations are [immutable](https://en.wikipedia.org/wiki/Immutable_object) - incremental changes are **not permitted**
- configuration is either fully replaced (`config replace`) via **TFTP** or the device is wiped clean when powered off and the new config is loaded via **DHCP** upon reboot
- rollbacks are simplfied with a single command (`git revert HEAD`)
- devices' **actual** state is continuously monitored and compared to the **desired** state
- alerts are generated if any configuration changes resulting in deviation from the desired state are detected
- GitHub documents the entire history of past changes (who did what, when, and why) and all team communication (pull requests, issue tracking, comments)


## Solution Components
- [GitHub Actions self-hosted runner](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners) - runs the workflow job
- [Ubuntu 20.04 Docker container](https://hub.docker.com/_/ubuntu) - hosts the GH Actions runner and the TFTP server
- [Opengear OOB access server](https://opengear.com/products/om2200-operations-manager/) - physical server running the Docker Engine
- [Unimus network automation tool](https://unimus.net/) - backs up and audits device configs
- Slack - sends config change notifications
- [Netmiko Python library](https://github.com/ktbyers/netmiko) - SSH into devices and replaces config


## Reference Architecture
![](/diagram-network-automation-github-actions.png)


## Usage
**Workflow steps**  
- *Standard* change
  - clone this repo
  - modify the device configuration files(s)
  - commit and push directly to the main branch
- *Normal* change
  - clone this repo, create a new branch and publish it
  - modify the device configuration files(s)
  - commit changes to the new branch and push to origin
  - create a pull request to submit proposed change(s)
  - pull request peer review
  - pre-deployment testing (functional/integration/performance) for complex and high risk changes
  - pull request approval and merge based on validation test results
- GitHub Actions workflow is triggered
  - workflows can also be triggered manually via GH CLI (`gh workflow run <WORKFLOW_NAME>`) or GUI from the repo's Actions page on GH
- self-hosted runner starts running the job(s)
- Python script is executed
- static testing (syntax check/config validation) by the device NOS
- devices' configuration is replaced
- Unimus continuously audits devices' operational state and generates alerts if config drift is detected
- operator gets a Slack notification describing the config change(s)


## Build
**Download and run Ubuntu on Opengear**
```
sudo -i
docker pull ubuntu
docker run -it ubuntu
```

**Install required packages**
```
apt update && apt upgrade -y

apt install apt-utils curl git python3-pip tftpd-hpa vim wget -y
apt install software-properties-common -y

pip3 install --upgrade pip keyring keyrings.alt netmiko
```

**Install [GitHub CLI](https://github.com/cli/cli/blob/trunk/docs/install_linux.md), login to GitHub and store GH credentials in git**
- *Create a Personal Access Token: github.com > profile pic > Settings > Developer settings > Personal access tokens > Generate new token: repo, read:org*
```
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key C99B11DEB97541F0
apt-add-repository https://cli.github.com/packages
apt update
apt install gh

gh auth login (GitHub.com > HTTPS > n > Paste an authentication token)

git config --global credential.helper store
```

**Create a new user and directories**
```
useradd -ms /bin/bash siteadmin && su siteadmin
mkdir /home/siteadmin/actions-runner && mkdir /home/siteadmin/actions-runner/network-automation-github-actions
```

**Store and encrypt device login credentials**
```
python3
import keyring
keyring.set_password('<SYSTEM_NAME>', '<USERNAME>', '<PASSWORD>')

# verify the password
keyring.get_password('cisco', 'siteadmin')

quit()
exit
```

**Configure and start the TFTP service as root**
```
cat /etc/default/tftpd-hpa
vi /etc/default/tftpd-hpa

TFTP_USERNAME="siteadmin"
TFTP_DIRECTORY="/home/siteadmin/actions-runner/network-automation-github-actions"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure"

/etc/init.d/tftpd-hpa start

# verify TFTP service is running
service --status-all
```

**Clone this repo**
```
su siteadmin && cd /home/siteadmin/actions-runner/
git clone https://github.com/gdmoney/network-automation-github-actions.git
```

**Download, extract, configure, and run the GitHub Actions self-hosted agent**
- *Get the runner version and the token from: github.com > repo > Settings > Actions > Add runner*  
```
curl -O -L https://github.com/actions/runner/releases/download/v<RUNNER_VERSION>/actions-runner-linux-x64-<RUNNER_VERSION>.tar.gz
tar xzf ./actions-runner-linux-x64-<RUNNER_VERSION>.tar.gz

./config.sh --url https://github.com/gdmoney/network-automation-github-actions --token <TOKEN>
./run.sh

Connected to GitHub
Listening for Jobs
```

**Uninstall the agent (optional)**
- *Get the token from: github.com > repo > Settings > Actions > ... > Remove*
```
./config.sh remove --token <TOKEN>
```
