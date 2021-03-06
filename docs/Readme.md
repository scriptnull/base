# General Information

## Supported command line arguments

The following cli is supported by the installer

```bash
$ ./base.sh --help

  usage: ./base.sh options
  This script installs Shippable enterprise
  OPTIONS:
    -s | --status     Print status of current installation
    -i | --install    Start a new Shippable installation
    -r | --release    Install a particular version
    -f | --file       Use existing state file
    -v | --version    Print version of this script
    -h | --help       Print this message
```

## Installation modes

The installer supports two modes of installation: `local` and `production`. 
These can be used via following cli statement

```
$ ./base.sh --install [local|production]
```

If no value is provided, `local` is considered by default

### `local` installation mode

This assumes that all Shippable components are being installed on a single machine, usually for 
local development. When run in `local` mode, installer does not install any underlying components like
`postgres`, `rabbitmq`, `redis` etc. Instead, it brings up docker containers for these components and
starts them with `net=host` option. This makes sure all the ports are mapped to the host and can be used
easily by other services.

To start installer in `local` mode, follow instructions [here](local.md).

### `production` installation mode

This mode assumes that all Shippable components are being installed on a topology of at least 4 servers.
The (recommended)composition of servers should be:  

- core: for database, rabbitmq and vault
- manager: for swarm master, gitlab, redis
- services-1: for running microservices
- services-2: for running microservices

To start installer in `production` mode, follow instructions [here](production.md).

## SSH Keys

The installer generates the ssh key-pair to be used for communicating with other machines in the topology. This
is a manual step where the user needs to run the command on each host to add the public ssh key of the `manager`
in the `authorized_keys` file of the hosts. Once the `manager` is able to ssh into the hosts, it'll set up 
rest of the components based on host configuration.

## State file

The state of the system is kept in a file at the location `usr/state.json`. This file contains the progress of
installation and all installation-specific data like keys to pull docker images, message queue credentials, 
proxy configuration, docker version for custom hosts and generated tokens. This will be used to recover the system
and for debugging purposes. This file should **NEVER** be deleted. Some values can be alterted manually but the user
has to be absolutely sure about what is being changed.   
In almost all cases, installer will take care of updating and backing up this file
More information on state file can be found [here](state.md)

## Load balancers

For `production` installation, Shippable requires three loadbalancers

- for accessing UI (should point to `services-1` and `services-2` machines)
- for accessing API (should point to `services-1` and `services-2` machines)
- for accessing Message Queue (should point to `core` machine)

These should be internet-routable (or be accessible within the VPN) and should be configured before the
installer is run.

## Troubleshooting

- to verify the json after any changes in `usr/state.json` run following command

```
$ cat usr/state.json | jq '.'
```

## FAQ

Q: What happens if I accidently terminate installer?  
A: As long as the state file (`usr/state.json`) is preset, installer can be run any number of times without any issues.

Q: Does installer install components like database, rabbitmq etc every time it runs?  
A: No, once the components are installed, installer never installs them again. It keeps track of installed components
in state file and every component has a flag asssociated with it. Only if that flag is reset, the installer will
re-install the component

Q: How do I enable more master integrations?  
A: Just add the required master integrations in `masterIntegrations` array in `usr/state.json` and run installer again.

Q: How do I disable master integrations?  
A: Remove the master integrations from `usr/state.json` and run installer again.

Q: Are there any templates for integrations?  
A: Yes, the templates for all the `masterIntegrations` and `systemIntegrations` are available in `usr/state.json.example`. These
can be copied over to `usr/state.json` as is. Once the required fields are updated, run installer again to sync these with database.
