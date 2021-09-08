# Ansible Role: Razor

An Ansible Role that installs Puppetlabs Razor on Linux.

## Razor Summary

PuppetLabs Razor Server is an Open Source Operating System provisioning tool, details are here:

[https://github.com/puppetlabs/razor-server/wiki]

## Installation Source

Instructions for installing Razor found here which this role is based off of including work required to get the requirements working.

[https://github.com/puppetlabs/razor-server/wiki/Installation]

## Razor Post Installation Configuration

This role will only install Razor and the default OS Tasks. I've created the following personal Roles for custom Tasks, Repos, Policies and Tags:

* nicholasrodriguez.razor_tasks_centos
* nicholasrodriguez.razor_tasks_esxi
* nicholasrodriguez.razor_manage_hosts

These can be copied and used as template for your own environments.

## Razor Manual Use Notes

The following are the Razor cli commands which have been replicated in the Ansible automation above but are here for reference. To PXE build a server you need to
1. Get the server serial number or MAC
2. Create a TAG in Razor
3. Create a Policy in Razor

### Get the Server Serial Number
1. On the server to be built ensure PXE Boot is enabled first and switch it on
2. The server should boot to the Razor CentOS7 micro kernel and wait
3. On the Razor run the following
```
razor nodes
```
4. Work out which node is the one just booted and run the following replacing the appropriate number
```
razor nodes node<NUMBER OF TARGET NODE> facts | grep serial number
```

### Create Tag
1. Run the following command replacing the serial number with the one found in the previous step.
```
razor create-tag --name <HOSTNAME> --rule '["in", ["fact", "serialnumber"],"VMware-56 4d cd 0b 9d 4d e2 ca-23 88 34 30 95 b8 0f 0f"]'
```

### Create Policy
1. Create the following file replacing the various variables in <> with ones specific to the server and save as <HOSTNAME>.json
```
{
"name": "<HOSTNAME>",
"repo": "CentOS-7.2-minimal",
"task": "centos/7",
"broker": "noop",
"enabled": true,
"hostname": "<HOSTNAME>",
"root_password": "<ROOT PASSWORD>",
"max_count": 1,
"tags": ["<HOSTNAME>"],
"node_metadata": {
    "key1": "value1",
    "keyN": "valueN",
    }
}
```
2. Run the following command to create the policy in Razor
```
razor create-policy --json <HOSTNAME>.json
```

# Requirements

In order to PXE build servers Razor requires the following supporting services:

* dhcp
* tftp (locally installed)
* Postgres

This Role was built and tested with the following Roles but doesn't preclude using others to deploy the same requirements:

* nicholasrodriguez.tftp
* nicholasrodriguez.dhcp
* geerlingguy.postgresql

# Role Variables

Available variables are listed below, along with default values (see defaults/main.yml):

Required Razor packages
```
razor_packages:
  - razor-server
  - ruby
```

Location of the tftp root directory
```
tftp_root_directory: "/var/lib/tftpboot"
```

Name of the Postgres database for Razor
```
razor_db_name: "razor"
```

Name of the Postgres database users for Razor
```
razor_db_user_name: "razor"
```

Name of the Postgres database user password for Razor
```
razor_db_user_pw: "razor"
```

Name of the Linux Razor service user
```
razor_service_user: "razor"
```

Linux Razor service user password
```
razor_service_user_pw: "razor"
```

Name of the account for full API access (Shiro configuration)
```
razor_api_admin: "razor"
```

Password for account for full API access
```
razor_api_admin_pw: "razor"
```

List of API roles and the scopes of the roles (Shiro configuration)
```
razor_api_roles:
  - role_name: "admin"
    role_scope: "*"
  - role_name: "user"
    role_scope: "query:*"
```

List of accounts for restricted API access (Shiro configuration)
```
razor_api_users:
  - api_user_name: "razor_api_user"
    api_user_pw: "razor"
    api_user_roles: "user"
```

Razor Microkernel location.
Sometime the original Puppetlabs Microkernel source can take 10 minutes to respond
but this file could be hosted internally
```
razor_microkernel_source: "http://pup.pt/razor-microkernel-latest"
```

iPXE legacy location
```
ipxe_legacy_source: "http://boot.ipxe.org/"
```

iPXE legacy file
```
ipxe_legacy_file: "undionly.kpxe"
```

iPXE UEFI location
```
ipxe_uefi_source: "http://boot.ipxe.org/"
```

iPXE UEFI file
```
ipxe_uefi_file: "ipxe.efi"
```

Razor client required GEMs
```
razor_client_gems:
  - name: "colored"
    version: "1.2"
  - name: "command_line_reporter"
    version: "4.0.0"
  - name: "mime-types"
    version: "1.25.1"
  - name: "multi_json"
    version: "1.13.1"
  - name: "razor-client"
    version: "1.1.0"
  - name: "rest-client"
    version: "1.6.9"
```

# Dependencies

None

# Example Playbook
```
- hosts: servers
  roles:
     - role: nicholasrodriguez.razor
```

License
-------

MIT

Author Information
------------------

- https://github.com/nicholasrodriguez/ (maintainer)
