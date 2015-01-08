Portable OpenStack Cloud Initialization Service
===============================================

The main goal of this project is to provide guest cloud initialization for
Windows and other operating systems.

The architecture of the project is highly flexible and allows extensions for
additional clouds and plugins.

There's no limitation in the type of supported Hypervisor. This service can be
used on instances running on Hyper-V, KVM, Xen, ESXi, etc

Documentation, support and contacts: http://www.cloudbase.it

Binaries
--------

The following x64 and x86 builds are automatically generated by a Jenkins job
at every commit:

https://www.cloudbase.it/downloads/CloudbaseInitSetup_Beta_x64.msi
https://www.cloudbase.it/downloads/CloudbaseInitSetup_Beta_x86.msi

Metadata services
-----------------

A metadata service has the role of pulling the metadata configuration
information.

Supported clouds and metadata services:

* OpenStack (HTTP)
* OpenStack (ConfigDrive)
* Amazon EC2
* CloudStack
* OpenNebula
* Ubuntu MAAS

Plugins
-------

Plugins execute actions based on the metadata obtained by the service.


#### cloudbaseinit.plugins.windows.sethostname.SetHostNamePlugin

Sets the instance's hostname.


#### cloudbaseinit.plugins.windows.createuser.CreateUserPlugin

Creates a local cloud user (if it does not already exist) and adds it to a set
of provided local groups.

The following configuration parameters control the behaviour of this
plugin.

| Option     | Description                    | Default          |
|------------|--------------------------------|------------------|
| _username_ | Name of the cloud user         | _Admin_          |
| _groups_   | Comma separated list of groups | _Administrators_ |


#### cloudbaseinit.plugins.windows.setuserpassword.SetUserPasswordPlugin

Sets the cloud user's password. If a password has been provided in the
metadata during boot (user_data) it will be used, otherwise a random password
will be generated, encrypted with the user's SSH public key and posted to the
metadata provider (currently supported only by the OpenStack HTTP metadata
provider).

| Option                 | Description                                                                         | Default |
|------------------------|-------------------------------------------------------------------------------------|---------|
| _inject_user_password_ | Can be set to false to avoid the injection of the password provided in the metadata | True    |


#### cloudbaseinit.plugins.windows.networkconfig.NetworkConfigPlugin

Configures static networking.

| Option            | Description                  | Default |
|-------------------|------------------------------|---------|
| _network_adapter_ | Network adapter to configure | _None_  |

If _network_adapter_ is not specified, the first available ethernet
adapter will be chosen if it cannot be matched with the configuration provided
in the metadata.


#### cloudbaseinit.plugins.windows.sshpublickeys.SetUserSSHPublicKeysPlugin

Creates an "authorized_keys" file in the user's home directory containing the
SSH keys provided in the metadata.
It is needed by the
_cloudbaseinit.plugins.windows.setuserpassword.SetUserPasswordPlugin_ plugin.


#### cloudbaseinit.plugins.windows.extendvolumes.ExtendVolumesPlugin

Extends automatically a disk partition to it's maximum size. This is useful
when booting images with different flavors.


#### cloudbaseinit.plugins.windows.winrmlistener.ConfigWinRMListenerPlugin

Configures a WinRM HTTPS listener to allow remote management via WinRS or
PowerShell.


#### cloudbaseinit.plugins.windows.winrmcertificateauth.ConfigWinRMCertificateAuthPlugin

Enables password-less authentication for remote management via WinRS or
PowerShell.
See: http://www.cloudbase.it/windows-without-passwords-in-openstack/


#### cloudbaseinit.plugins.windows.localscripts.LocalScriptsPlugin

Executes any script (e.g. Powershell, CMD, etc) located in the following path.

| Option               | Description        | Default |
|----------------------|--------------------|---------|
| _local_scripts_path_ | Local scripts path | _None_  |


#### cloudbaseinit.plugins.windows.licensing.WindowsLicensingPlugin

Activates the Windows instance if the following option is True.

| Option             | Description      | Default |
|--------------------|------------------|---------|
| _activate_windows_ | Activate Windows | _False_ |


#### cloudbaseinit.plugins.windows.ntpclient.NTPClientPlugin

Applies NTP client info based on the DHCP server options, if available.

| Option                | Description       | Default |
|-----------------------|-------------------|---------|
| _ntp_use_dhcp_config_ | Set NTP from DHCP | _False_ |


#### cloudbaseinit.plugins.windows.mtu.MTUPlugin

Sets the network interfaces MTU based on the value provided by the DHCP server
options, if available.

This is particularly useful for cases in which a lower MTU value is required
for networking (e.g. OpenStack GRE Neutron Open vSwitch configurations).

| Option                | Description       | Default |
|-----------------------|-------------------|---------|
| _mtu_use_dhcp_config_ | Set MTU from DHCP | _True_  |


#### cloudbaseinit.plugins.windows.userdata.UserDataPlugin

Executes custom scripts provided with the user_data metadata as plain text or
compressed with Gzip.

Supported formats:

##### Windows batch

The file is executed in a cmd.exe shell (can be changed with the COMSPEC
environment variable). The user_data first line must be:

    rem cmd

##### PowerShell

The user_data first line must be:

    #ps1_sysnative

or for a x86 PowerShell execution:

    #ps1_x86

##### Bash

A bash shell needs to be installed in the system and available in the PATH in
order to use this feature. The user_data first line must start with:

    #!

e.g.:

    #!/bin/bash

#### cloud-config

Cloud-config YAML configuration as supported by cloud-init, excluding Linux
specific content. The user_data first line must be:

    #cloud-config

Note: currently only local file creation is supported.


#### Multi-part userdata content

MIME multi-part userdata is supported. The content will ne handled based on the
content type.


##### text/x-shellscript

Any script to be executed: PowerShell, CMD, Bash or Python.


##### text/part-handler

A script that can manage other content type parts. This is used in particular
by Heat / CFN templates, although Linux specific.

##### text/x-cfninitdata

Heat / CFN content. Written to the path provided by:

| Option            | Description             | Default     |
|-------------------|-------------------------|-------------|
| _heat_config_dir_ | Heat configuration path | _C:\\cfn_   |

Example Heat Windows templates: https://github.com/openstack/heat-templates/tree/master/hot/Windows
