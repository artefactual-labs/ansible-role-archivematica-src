archivematica-src
=================

Archivematica installation from its source code repositories.

**Table of Contents**

- [Role Variables](#role-variables)
- [Environment variables](#environment-variables)
- [Backward-compatible logging](#backward-compatible-logging)
- [Tags](#tags)
- [Dependencies](#dependencies)
- [Example Playbooks](#example-playbooks)
- [License](#license)
- [Author Information](#author-information)


Role Variables
--------------

See [`defaults/main.yml`](defaults/main.yml) for a comprehensive list of variables.


Environment variables
---------------------

The following are role variables that can be used to pass dictionaries containing environment variables that will be passed to the different Archivematica components:

- [Dashboard](https://github.com/artefactual/archivematica/tree/stable/1.7.x/src/dashboard/install/README.md): `archivematica_src_am_dashboard_environment`
- [MCPServer](https://github.com/artefactual/archivematica/tree/stable/1.7.x/src/MCPServer/install/README.md): `archivematica_src_am_mcpserver_environment`
- [MCPClient](https://github.com/artefactual/archivematica/tree/stable/1.7.x/src/MCPClient/install/README.md): `archivematica_src_am_mcpclient_environment`
- [Storage Service](https://github.com/artefactual/archivematica-storage-service/tree/stable/0.11.x/install/README.md): `archivematica_src_ss_environment`

The default values for these dictionaries can be found in [`vars/envs.yml`](vars/envs.yml). The user-provided dictionaries are combined with the defaults.

The authoritative place to find accurate information about the environment variables supported is the install/README.md file for each Archivematica component. Use the links above to find them.

In the following example we're going to redefine the environment variables of the MCPServer.

```yaml
---
archivematica_src_am_mcpserver_environment:
  ARCHIVEMATICA_MCPSERVER_MCPSERVER_SHAREDDIRECTORY: "/tmp/shared-directory"
```

The final environment generated by this role is:

```json
{
  "DJANGO_SETTINGS_MODULE": "settings.common",
  "ARCHIVEMATICA_MCPSERVER_MCPSERVER_SHAREDDIRECTORY": "/tmp/shared-directory"
}
```

There are a number of role variables that have an effect in the final environment dictionaries, e.g. when `archivematica_src_ca_custom_bundle` is used the environment string `REQUESTS_CA_BUNDLE` is added to all the environment dictionaries. This is mostly done for backward-compatibility reasons or for convenience. See [`tasks/envs-patch-backward-compatibility.yml`](tasks/envs-patch-backward-compatibility.yml) for a full list of variables available and its effects.

Backward-compatible logging
---------------------------

The default Archivematica 1.7 logging sends the events to the standard streams, which is more convenient when Archivematica is running in a cluster. In order to use the backward-compatible logging, the boolean environment variable `archivematica_src_logging_backward_compatible` has to be enabled, which is the default behaviour.

The log file sizes and the directories to store the logs are configurable for each service. The default values can be found in [`defaults/main.yml`](defaults/main.yml).

Configure ClamAV
----------------

This role will try to determine whether the ClamAV daemon is running using the TCP or UNIX socket on the same server that the pipeline is being installed or updated on. To configure an external ClamAV daemon server, the following env vars should be set: 

```yaml
---
archivematica_src_mcpclient_clamav_use_tcp: "yes"
archivematica_src_mcpclient_clamav_tcp_ip: "1.2.3.4"
archivematica_src_mcpclient_clamav_tcp_port: "3310"
```

Disable Elasticsearch use
-------------------------

The default Archivematica install relies on Elasticsearch for different features (Archival storage, Backlog and Appraisal tabs). If you need to disable them, the role variable `archivematica_src_search_enabled` has to be set to `False`

Tags
----

Tags can be used to control which parts of the playbook to run, especially on updates.
Note that if something is disabled with the [role variables](#role-variables), it won't be run even if the tag is enabled.

- `amsrc-ss`: Storage service install
    - `amsrc-ss-clone`: Checkout source code
    - `amsrc-ss-osdep`: Install operating system dependencies
    - `amsrc-ss-pydep`: Install Python dependencies (with pip)
    - `amsrc-ss-osconf`: Configure operating system
    - `amsrc-ss-code`: Install source code
        - `amsrc-ss-code-collectstatic`: Run Django's collectstatic
    - `amsrc-ss-db`: Configure database
    - `amsrc-ss-websrv`: Configure webserver
- `amsrc-pipeline`: Archivematica pipeline install
    - `amsrc-pipeline-clonecode`: Checkout source code
    - `amsrc-pipeline-deps`: Install & configure operating system & Python dependencies
    - `amsrc-pipeline-osconf`: Configure operating system
    - `amsrc-pipeline-instcode`: Install source code
    - `amsrc-pipeline-dbconf`: Configure database
        - `amsrc-pipeline-dbconf-syncdb`: Only run Django's syncdb/migrations
    - `amsrc-pipeline-websrv`: Configure webserver
- `amsrc-devtools`: Archivematica devtools install
- `amsrc-automationtools`: Automation tools install
- `amsrc-configure`: Create SS superuser & create dashboard admin & register pipeline on SS


Dependencies
------------

Role dependencies are listed in `meta/main.yml`. 

Notes regarding role dependencies:

  - `geerlingguy.nodejs`: if having an error on task `Create npm global directory`, please set the `nodejs_install_npm_user` to  `{{ ansible_user_id }}` as a workaround (details in https://github.com/artefactual-labs/ansible-archivematica-src/issues/151)


Please use Ansible 2.3 or newer with this role.


Example Playbooks
-----------------

Please note that a complete Archivematica installation includes software not installed by this role, in particular:

- MySQL compatible database server (MySQL, MariaDB, Percona)
- Elasticsearch
- ClamAV (daemon and client)

See https://github.com/artefactual/deploy-pub/tree/master/playbooks/archivematica to find examples.

It is also recommended to take backups of your system (Archivematica and Storage Service databases, AIPS, DIPS, etc) prior to running an upgrade.


License
-------

AGPLv3


Author Information
------------------

Artefactual Systems Inc.
https://www.artefactual.com
