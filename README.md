# rockon-registry

This repository consists of Rock-on (Docker based apps) configuration profiles
formatted as JSON files. The Rock-on framework of Rockstor parses a well
formatted profile and provides a generic management and workflow such as
install, uninstall, update, start and stop.

## Can You Show Me an Example??

Look at any <name>.json file in this repository. A simpler example is
[syncthing.json](https://github.com/rockstor/rockon-registry/blob/master/syncthing.json). The structure is fairly intuitive though cumbersome. Using existing examples and the description below of the `json` structure should make it clearer on how this framework is organized.

## How Can I Add My Own Rock-on?

If you are familiar with Docker and know how to deploy docker containers using the command line, you can create a Rock-on for the same with little effort.
You can find instructions in the [Rockstor Documentation](https://rockstor.com/docs/contribute/contribute_rockons.html) both for running your own Rock-on locally, as well as how to contribute to the existing public repository of available Rock-ons

## What Is the Structure of a Rock-on Profile File?

It is a big mass of JSON with nested objects, arrays and values.

### Top Level Structure
```
{
    "<Rock-on name. eg: LSIO-Plex>": {
      "description": "<description of the Rock-on. Eg: Plex brought to you by Linuxserver.io>",
      "version": "<arbitrary version string>",
      "website": "<Underlying app website>",
      (optional)"icon": "<link to icon, if any>",
      (optional)"more_info": "<string or html with more information to display to the user in the Rockstor UI>",
      (optional)"ui":{
                "slug":"gui", link to webui becomes ROCKSTOR_IP:PORT/gui with slug value gui
      },
      (optional)"volume_add_support": true, If the app allows arbitrary Shares to be mapped to the main container>,
      "containers": {
        "<container1 name>": <container object representing the main container. see below>,
        "<container2 name>": <container object representing the second container, if any. see below>, ...
      },
      (optional)"custom_config": <custom configuration object that a special install handler of this Rock-on expects>
    }
}
```
_N.B. on the container names. If you are adding a net new Rockon that is a newer version of an existing one, or in a multi-container scenario uses a container for which a Rockon already exists (e.g. mariadb), ensure that the container name in your new Rockon is slightly different, otherwise a duplicate error will be thrown when the final json file is imported._
### Structure of a Container Object

Each container object is key'd by its name and nested within "containers" of the top level structure above. A typical container object has the following structure

```
{
  "image": "<docker image. eg: linuxserver/plex>",
  (optional)"tag": "tag of the docker image, if any. latest is used by default.>",
  "launch_order": "1 or above. If there are multiple containers and they must be started in order, specify here.>",
  (optional)"ports": {
    "<container side port number1>": <port object represending a port mapping between host and container. see below>,
    "<port number2>": <another port object, if necessary. see below>, ...
  },
  (optional)"volumes": {
    "<path1 inside container>": <volume object representing a Share<->directory mapping in the container. see below>,
    "<path2 inside container>": <another volume object, if necessary. see below>, ...
  },
  (optional)"opts": [ An array of option objects that represent container options such as --net=host etc.. see below],
  (optional)"cmd_arguments": [ An array of cmd_arguments objects that represent arguments to pass to the 'docker run' command. See below],
  (optional)"environment": {
    "<env var1 name>": <env object representing one environment variable required by this container. see below>,
    "<env var2 name>": <another env object, if necessary. see below>, ...
  },
  (optional)"devices": {
    "<device1 name>": <device object representing one device to be passed to this container. see below>,
    "<device2 name>": <another device object, if necessary. see below>, ...
  }
}
```

As it is evident from above, a container object has nested objects for port and volume mappings, container options, command arguments, and environment variables. These are described below.

### Port Object

```
{
  "description": "<A detailed description of this port mapping, why it's for etc..>",
  "label": "<A short label for this mapping. eg: Web-UI port>",
  "host_default": <integer: suggested port number on the host. eg: 8080>,
  (optional)"protocol": "<tcp or udp>",
  (optional)"ui":true,  not needed if false
}
```
Note that protocol is optional and if it's not present, both tcp and udp ports are mapped simultaneously. So if you wish to allow both tcp and udp, just don't specify protocol in the Port object.

### Volume Object

```
{
  "description": "<A detailed description. Eg: This is where all incoming syncthing data will be stored>",
  "label": "<A short label. eg: Data Storage>",
  (optional)"min_size": <integer: suggested minimum size of the Share, in KB>
}
```

### Options Object

An options object is a list of exactly two elements. (This needs to be improved or deprecated in favor of more specific design.)

`--net=host` would be represented as:

```
["--net", "host"]
```

Note that the opts field is a 2-d array, so the complete line for the above example looks like

```
"opts": [ ["--net", "host"] ],
```

### Command Arguments Object

A command arguments object is a list of exactly two elements detailing specific arguments to be passed onto the `docker run` command. As these arguments will simply be appended to the `docker run` command, they need to follow the same syntax and order. For instance, 

`docker run <...> image/name argument1 argument2="text2"` would be represented as:

```
["argument1", "argument2="text2"]
```

Note that, as for the options object, the cmd_arguments field is a 2-d array, so the complete line for the above example looks like

```
"cmd_arguments": [ ["argument1", "argument2="text2"] ],
```

### Environment Object

```
{
  "description": "<Detailed description. Eg: Login username for Syncthing UI>",
  "label": "Web-UI username",
  (optional)"index": <integer: 1 or above. order of this environment variable, if relevant>
}
```


### Devices Object

This optional object allows to pass a specific device to the Rock-on, and similarly to the Environment object, must have a "description" and "label".  
```
{
  "description": "<Detailed description of the device and its intent or specificities. Eg: path to device (/dev/xxx)>",
  "label": "Hardware encoding device",
  (optional)"index": <integer: 1 or above. order of this environment variable, if relevant>
}
```
Note that for the user, filling the fields corresponding to this object during the Rock-on installation is optional, allowing the users to leave the fields blank if not applicable to them.  
