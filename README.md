# Workshop: Building MCP Applications with Spring AI

## 1. Setup

### 1.1 Install multipass
#### 1.1.1 Ubuntu
On Ubuntu `multipass` may be installed as a snap:
```
sudo snap install multipass
```

#### 1.1.2 Debian
Install snapd using instructions [here](https://snapcraft.io/docs/tutorials/install-the-daemon/debian/#tutorials-install-the-daemon-debian).
```
sudo apt update
sudo apt install snapd
```
Then install the `multipass` snap:
```
sudo snap install multipass
```

#### 1.1.3 Fedora/RHEL
Please follow the instructions at [here](https://snapcraft.io/install/multipass/fedora) to enabled `snapd` and install multipass
as a snap:

```
sudo dnf install snapd
sudo ln -s /var/lib/snapd/snap /snap
sudo snap install multipass
```
#### 1.1.4 Windows
Download [multipass-1.17.1+win-win64.msi](https://github.com/canonical/multipass/releases/download/v1.17.2/multipass-1.17.2+win-win64.msi) from the Multipass [releases page](https://github.com/canonical/multipass/releases/latest). 
Run the installer (administrator rights required). Open a new terminal afterwards, so that multipass is on your PATH.

#### 1.1.5 macOS
Download [multipass-1.17.1+mac-Darwin.pkg](https://github.com/canonical/multipass/releases/download/v1.17.2/multipass-1.17.2+mac-Darwin.pkg) from the Multipass [releases page](https://github.com/canonical/multipass/releases/latest) and run the installer package.

### 1.2 Install nodejs and npm
#### 1.1.1 Ubuntu
#### 1.1.2 Debian
#### 1.1.3 Fedora/RHEL
#### 1.1.4 Windows
#### 1.1.5 macOS

### 1.3 Preparing an Ubuntu 26.04 virtual machine
#### 1.3.1 Launch a multipass VM
#### 1.3.2 Install GLM 4.7
#### 1.3.3 Install OpenJDK 21

## Implementing an MCP server

## Extending a chat-client for MCP
