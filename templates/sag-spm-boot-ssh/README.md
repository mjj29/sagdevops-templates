# Bootstrapping Platform Manager on remote UNIX machines

This template allows to install Platform Manager (SPM) on remote
UNIX hosts using SSH protocol.

## Requirements

### Supported Software AG releases

* Command Central 10.2 Fix2 or later
* Platform Manager 9.8 or later

### Supported platforms

* All UNIX platforms for which Command Central bootstrap installers are available for download from Empower

> NOTE: Windows platforms are NOT supported by this template. See [sag-spm-boot-winrm](../sag-spm-boot-winrm/README.md)

### System requirements for the target UNIX machines

* SSH daemon with SHA1 support enabled
* User account, e.g. sagadmin, with remote SSH login privileges and write access to the target installation directory
* Credentials for the remote connection user account, e.g. sagadmin, are configured. See [sag-cc-creds-dev](../sag-cc-creds-dev/README.md).

### System requirements for Command Central machine

* Any supported UNIX or Windows operating system
* Must have Command Central bootstrap installer for the target UNIX platform (*.sh file) saved in `CC_HOME\profiles\CCE\data\installers` folder. Verify by running:

```bash
sagcc list provisioning bootstrap installers
```

## Running as a standalone Composite Template

Bootstrap SPM 10.2 into `/home/vmtest/sag102` installation directory listening on port 8292 on
bgcctbp11 and bgcctbp12 Linux hosts. The remote SSH connection is authenticated with pre-configured
`VMTEST` username/password credentials.

```bash
sagcc exec templates composite apply sag-spm-boot-ssh nodes=bgcctbp11,bgcctbp12 \
  cc.installer=cc-def-10.2-fix1-lnxamd64.sh \
  install.dir=/home/vmtest/sag102 \
  spm.port=8292 \
  os.credentials.key=VMTEST \
  --sync-job --wait 360
```

Authenticate SSH connection using pre-configured `SSH-PRIVATE-KEY` credentials which point
to `~/.ssh/id_rsa` private key on CCE host for the user account that runs CCE, e.g. sagadmin:

```bash
sagcc exec templates composite apply sag-spm-boot-ssh nodes=bgcctbp11,bgcctbp12 \
  cc.installer=cc-def-10.2-fix1-lnxamd64.sh \
  install.dir=/home/vmtest/sag103 \
  spm.port=8392 \
  os.credentials.key=SSH-PRIVATE-KEY \
  --sync-job --wait 360
```

## Adding UNIX Infrastructure layer to a Stack using CLI

Create a new 10.1 Dev01 stack and provision LinuxInfra infrastructure layer onto host1 and host2.
The SSH connection from CCE to host1 and host2 is authenticated via the sagadmin user account private key.

```bash
sagcc create stacks alias=Dev01 release=10.1
sagcc create stacks Dev01 layers alias=LinuxInfra layerType=INFRA-REMOTE-UNIX nodes=host1,host2 \
  cc.installer=cc-def-10.1-fix8-lnxadm64.sh \
  install.dir=/opt/softwareag \
  os.credentials.key=SSH-PRIVATE-KEY \
  --sync-job --wait 360
```

See [sag-cc-layer-defs](../sag-cc-layer-defs/template.yaml) for `INFRA-REMOTE-UNIX` layer type definition.

## Creating a new Stack with UNIX infrastructure layer using Web UI

* Open Stacks UI
* Add new stack
* Add layer > New nodes
  * Select INFRA-REMOTE-UNIX layer definition
  * Choose the operating system (except Microsoft Windows x86-64), and a corresponding Bootstrapper
  * Provide required parameters such as
    * os.credentials.key - SSH credentials
    * nodes - one or more host names
    * install.dir - remote installation directory
  * Finish the wizard
* Wait until provision jobs completes. Use Jobs view to monitor

## Local testing

You can test the template against `localhost` on Linux or Mac OS machine if it has SSHD running.
For example:

```bash
sagcc exec templates composite apply sag-spm-boot-ssh nodes=`hostname` \
  cc.installer=cc-def-10.2-fix1-osx.sh \
  install.dir=$HOME/test4 \
  spm.port=8492 \
  os.credentials.key=SSH-PRIVATE-KEY \
  --sync-job --wait 600
```