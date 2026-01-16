# rockon-registry

This repository consists of Rock-on (Docker based apps) configuration profiles formatted as JSON files.
The Rock-on framework of Rockstor parses a well formatted profile and provides a generic management and workflow such as install, uninstall, update, start and stop.

## Can you show me an example??

Look at any <name>.json file in this repository.
A simpler example is [syncthing.json](https://github.com/rockstor/rockon-registry/blob/master/syncthing.json).
The structure is fairly intuitive though cumbersome.
Using existing examples and the description below of the `json` structure should make it clearer on how this framework is organized.

## How can I add my own Rock-on?

If you are familiar with Docker and know how to deploy docker containers using the command line, you can create a Rock-on for the same with little effort.
You can find instructions in the [Rockstor Documentation](https://rockstor.com/docs/contribute/contribute_rockons.html) both for running your own Rock-on locally, as well as how to contribute to the existing public repository of available Rock-ons

## What is the structure of a Rock-on profile file?

It is a big mass of JSON with nested objects, arrays and values.

### Format & Norms

We have a developing standard regarding normalisation of the Rock-on format.
This is progressively being encoded into our developer utility know as the
[Rockon-validator](https://github.com/rockstor/rockon-validator).
When preparing a new Rock-on, consider using the rockon-validator tool to ensure compliance.

Example Expectations:
- JSON indent by 2 spaces.
- The **Rock-on name** (top-level element) should use
  [Title Case](https://apastyle.apa.org/style-grammar-guidelines/capitalization/title-case).
  Whilst honouring mark capitalisation such as in "ESPHome" & "2FAuth".
- **description & more_info** values must be in sentence or multi sentence form.
- **description** (top level) must contain either:
- - "... available for amd64 architecture only." or
- - "... available for amd64 and arm64 architecture.".
- **description** (top level) can contain for example: "Rockstor V5.5.0 or later." to indicate minimum compatibility.
- **description** (top level) indicate any external requirement e.g. "3rd-Party / Cloud account required|optional".
- **uid, gid, & launch_order** values are integer only; i.e. no quotes.
- **label** values for *Shares* and *Ports* must end with `[e.g. runner-data]` or `[must be 9100]`.
- **environment** labels should, if possible, provide guidance: i.e. end with `[e.g. Rockstor's IP]` or `[see Email Alerts]`.
- The **root.json** file is alphabetically sorted by the key elements (lower-case **Rock-on names**).  

*The above "Norms" are [in-developemnt](https://github.com/rockstor/rockon-registry/issues/465).*

**Each subsection below also details further expectations.** 

### Top level structure
```
{
    "<Rock-on name. e.g. `LSIO-Plex`>": {
      "description": "<description of the Rock-on. E.g. `Plex brought to you by Linuxserver.io`>",
      "version": "<arbitrary version string>",
      "website": "<Underlying app website>",
      (optional)"icon": "<link to icon, if any>",
      (optional)"more_info": "<string or html with more information to display to the user in the Rockstor UI>",
      (optional)"ui":{
                "slug":"gui", link to webui becomes ROCKSTOR_IP:PORT/gui with slug value gui
      },
      (optional)"volume_add_support": <true, If the app allows arbitrary Shares to be mapped to the main container>,
      (optional)"container_links": { <network object representing links between containers using separate docker networks, See below.>
      },
      "containers": {
        "<container1 name>": <container object representing the main container. See below.>,
        "<container2 name>": <container object representing the second container, if any. See below.>, ...
      },
      (optional)"custom_config": <custom configuration object that a special install handler of this Rock-on expects>
    }
}
```

### Structure of the container_links object

The basic structure of the optional `container_links` object is:

```
{
  "<target container": [
    {
      "name": "<Name of new docker network 1 to be created",
      "source_container": "<source container 1 that should connect to target container>"
    },
    {
      "name": "<Name of new docker network 2 to be created",
      "source_container": "bareos-storage"
    },
    {
      ...
    }
  ]
}
```

_N.B. Current restrictions: a source/destination/network combination is considered a unique entry:_

_A destination container can have multiple sources, but each of these connections will generate its own docker network.
This essentially provides only a point-to-point connection._

_In future releases this constraint will be removed, so, like with Rocknets, the network becomes the center and any container defined in the Rockon can be connected to it,
allowing for multi-directional communication._

### Structure of a `container` object

_N.B. on the container names.
If you are adding a net new Rockon that is a newer version of an existing one, or in a multi-container scenario uses a container for which a Rockon already exists (e.g. mariadb),
ensure that the container name in your new Rockon is slightly different,
otherwise a duplicate error will be thrown when the final json file is imported._

Each container object is keyed by its name and nested within "containers" of the top level structure above.
A typical container object has the following structure; elements detailed later as required.

```
{
  "image": "<docker image. eg: linuxserver/plex>",
  (optional)"tag": "<tag of the docker image, if any. "latest" is used by default.>",
  "launch_order": "<integer: 1 or above. If there are multiple containers and they must be started in order, specify here.>",
  (optional) "uid": <integer: for `--user uid` container execution: users default groups are implied.>,
  (optional) "gid": <integer: for `--user uid:gid` container execution: ignored if no uid is defined.>,
  (optional)"ports": {
    "<container side port number1>": <port object represending a port mapping between host and container.>,
    "<port number2>": <another port object, if necessary.>, ...
  },
  (optional)"volumes": {
    "<path1 inside container>": <volume object representing a Share<->directory mapping in the container.>,
    "<path2 inside container>": <another volume object, if necessary.>, ...
  },
  (optional)"opts": [ An array of option objects that represent container options such as --net=host etc.],
  (optional)"cmd_arguments": [ An array of cmd_arguments objects that represent arguments to pass to the 'docker run' command. See below.],
  (optional)"environment": {
    "<env var1 name>": <env object representing one environment variable required by this container.>,
    "<env var2 name>": <another env object, if necessary.>, ...
  },
  (optional)"devices": {
    "<device1 name>": <device object representing one device to be passed to this container.>,
    "<device2 name>": <another device object, if necessary.>, ...
  }
}
```

### `uid` & `gid` elements

These optional elements control the `--user` directive in the `docker run` command for the given container.
The `gid` element depends on the existence of a `uid` element and requires **Rockstor 5.5.0-0 onwards.**

Unless a container image was specifically developed to run with a non-root user, it will run as the root user initially.
This is often required to enable such containers to change their own UID or GID via environmental variables,
if they offer this option.
Otherwise, a container's own USER uid:gid (Dockerfile) directives are directly mapped by UID:GID not name,
to the host system.

The `--user` directive overrides the UID (default group/s) or UID:GID (specific GID) for container execution;
enabling desired user, or user:group mapping to, for example, Share ownership.
I.e. `--user` (via the `uid` & `gid` Rock-on elements) overrides Dockerfile defaults for the USER directive.
E.g. a container has an internal user named "runner" (USER runner) setup beforehand within the container with UID=1000.
Docker maps this container user to the host user (Rockstor user) with the same UID.
Rockstor's UID 1000 user is normally created during the
[Rockstor Setup and EULA](https://rockstor.com/docs/installation/installer-howto.html#rockstor-setup-and-eula) stage.
** I.e. an unprivileged linux user with UID=1000 & GID=100 (1000:100). ** 

For a Rockstor user's UID, see the SYSTEM -> `Users` page within in the WebUI.
Or run `id <username>` at the command line for both UID and GID information.

The available combinations are:
- No `uid` element: no `--user` directive/override is used; irrespective of `gid` value.
- `uid: -1`: run as the owner of the first `volume` object / Share.
- `gid: -1`: run as the group of the first `volume` object / Share. N.B. requires a `uid` entry.
- `gid: -2`: run as the **docker** host GID. N.B. requires a `uid` entry. **Rockstor 5.5.0-0 onwards.**

A `container` object also has nested objects for;
ports, volume mappings, container options, command arguments, and environment variables.
These are described below.

### `ports` object
This optional object can be used for mapping of external ports onto the ports published by the container image.

```
{
  "description": "<A detailed description of this port mapping, what it is for, etc.>",
  "label": "<A short label for this mapping. e.g. Web-UI port>",
  "host_default": <integer: suggested port number on the host. e.g. 8080>,
  (optional)"protocol": "<tcp or udp>",
  (optional)"ui":true,  not needed if false
}
```

_Note that protocol is optional and if it's not present, both tcp and udp ports are mapped simultaneously.
If you wish to allow both tcp and udp, just don't specify protocol in the Port object._

### `volumes` object
This optional object allows the mapping of Rockstor Shares onto volume paths exposed by the container image.
When Shares require, or may benefit from a specific owner:group,
specify this at the end of the Share's *"description"* via name or ID,
e.g. "... must be (1000:docker)." or "... e.g. (13001:13001)." or "... (bareos:bareos)".
Docker can only map IDs irrespective, but with for example (bareos:bareos),
the docs instruct on creating a host to in-container uid:gid mapping.
```
{
  "description": "<A brief description. E.g. 'This is where all incoming syncthing data will be stored (1000:1000).'>",
  "label": "<A short label, e.g. 'Data Storage [e.g. syncthing-storage]'>",
  (optional)"min_size": <integer: suggested minimum size of the Share, in Bytes>
}
```

### `options` object

An (optional) `options` object is a list of exactly two elements (This needs to be improved or deprecated in favor of a more specific design).

`--net=host` would be represented as:

```
["--net", "host"]
```

Note that the `opts` field is a 2-d array, so the complete line for the above example looks like this:

```
"opts": [ ["--net", "host"] ],
```

### `cmd` object

The optional `cmd` (command arguments) object is a list of exactly two elements detailing specific arguments to be passed to the `docker run` command itself.
As these arguments will simply be appended to the `docker run` command, they need to follow the same syntax and order.
For instance, 

`docker run <...> image/name argument1 argument2="text2"` would be represented as:

```
["argument1", "argument2="text2"]
```

Note that, as for the `options` object, the `cmd_arguments` field is a 2-d array, so the complete line for the above example appears like this:

```
"cmd_arguments": [ ["argument1", "argument2="text2"] ],
```

### `environment` object
This optional object allows to pass environment variables that are exposed by the container image into the container.

```
{
  "description": "<Detailed description. E.g. 'Login username for Syncthing UI'>",
  "label": "Web-UI username",
  (optional)"index": <integer: 1 or above. Order of this environment variable, if relevant.>
}
```


### `devices` object

This optional object allows to pass a specific device to the Rock-on, and similarly to the Environment object, must have a "description" and "label".  

```
{
  "description": "<Detailed description of the device and its intent or specificities. E.g. 'path to device (/dev/xxx)'>",
  "label": "Hardware encoding device",
  (optional)"index": <integer: 1 or above. Order of this environment variable, if relevant>
}
```
Note that for the user, filling the fields corresponding to this object during the Rock-on installation is optional, allowing the users to leave the fields blank if not applicable to them.  
