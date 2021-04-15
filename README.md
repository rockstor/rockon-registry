# rockon-registry

This repository consists of Rock-on(Docker based apps) configuration profiles
formatted in JSON files. The Rock-On framework of Rockstor parses a well
formatted profiles and provides a generic management and workflow such as
install, uninstall, update, start and stop.

## Can you show me an example?

Look at any <name>.json file in this repository. A simpler example is
[syncthing.json](https://github.com/rockstor/rockon-registry/blob/master/syncthing.json). The structure is intuitive(though cumbersome) and with the help of examples and below description, you can add your own apps with some effort.

## How can I add my own app?

If you are familiar with Docker and know how to run apps by hand, you can create a Rock-on for the same with a little bit of craft. There are three broad steps.

1. Configure the Rock-On service on your Rockstor system. Follow this [doc](http://rockstor.com/docs/docker-based-rock-ons/overview.html).

2. Create your Rock-on profile file, [app].json following the clues in this readme.

3. Upload the file to /opt/rockstor/rockons-metastore/[app].json. Hit update in the Web-UI and install your brand new Rock-On!

If you like to share your app with rest of the Rockstor community, open a pull request in this repository. Please follow these guidelines when opening a pull request.

1. One Rock-On per pull request please. If you are working on multiple apps, separate them out. It will make testing and merging a lot more manageable.

2. Add a comment to your pull request detailing how you've tested it out. More details the better as it will help ensure quality and benefit the whole community!

3. We are trying to offer Rock-ons that are based on a multi-architecture docker image, _i.e._, it is available for the amd64 (Intel/AMD CPUs), armv7 and arm64 (ARM-based devices). While that might not always be possible, depending on the app in which you're interested, please keep that in mind and see whether you can select a docker image that has a multi-architecture manifest. When submitting, please add the supported architectures to the description (take a look at some of the more recent Rock-on submissions).

## Adding vs. updating a Rock-on you want to submit to the repository

While there are no hard and fast rules, you can follow these guidelines to decide whether it is better to update an existing Rock-on or submit a new one that contains substantial changes:
1. If no Rock-on exists for your app already, create a new one. The obvious choice.
2. Create a new Rock-on if one already exists for this project, but the new one will use a different docker image. E.g., an existing Rock-on uses an image from linuxserver.io, while the one you are interested in submitting uses an image created by the owner of the project. You can subsequently submit a proposal to deprecate the existing Rock-on. Reasons for that could be that the underlying docker container has not been maintained in a long time, or the new Rockon will have the same or more functionality and is more popular with the community.
3. Update the existing Rock-on if the changes do not include the use of a different image. E.g., the Handbrake Rock-on was expanded with a few useful user parameters over time, but continued to use the same underlying docker image. Here it made more sense to update the existing Rock-on instead of submitting a new version.

## One time setup before opening pull requests.

Go to [rockon-registry repo](https://github.com/rockstor/rockon-registry) and click on the <b>Fork</b> button. This will fork the repository into your profile which serves as your private git remote called origin. The next few git steps are demonstrated on a Linux terminal. 

Create a local clone of your fork.
```
git clone git@github.com:your_github_username/rockon-registry.git
```
Configure this new git repo with your name and email address. This is required to accurately record collaboration.
```
cd rockon-registry
git config user.name "Firstname Lastname"
git config user.email your_email_address
```

Add a remote called upstream to periodically rebase your local repository with changes in the upstream made by other contributors.
```
git remote add upstream https://github.com/rockstor/rockon-registry.git
```
Above 4 steps help you setup your local environment. If you are familiar with git and use an IDE like Eclipse, you can achieve the same outcome in a different way. Here, we listed the simple terminal way of setting it up.

## Steps to Contribute a Rock-on with a pull request

Rebase your master branch before making your own changes.
```
cd rockon-registry
git checkout master
git pull --rebase upstream master
```

Checkout a new/separate branch for your Rock-on
```
git checkout -b rockon_name
```

Add and commit your rockon to git. Say you are working on Syncthing Rock-on and have the syncthing.json tested and ready to go. First copy the file over to your repo. Next,
```
git add syncthing.json
git commit -m 'add synctthing rock-on'
```

Push your rockon to github. <branch_name> is from two steps ago.
```
git push origin <branch_name>
```

Now go to github and open a pull request. Thanks!

## What is the structure of a Rock-On profile file?

it's a big mass of JSON with nested objects, arrays and values.

### Top level structure
```
{
    "<Rock-On name. eg: LSIO-Plex>": {
      "description": "<description of the Rock-On. Eg: Plex brought to you by Linuxserver.io>",
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
### Structure of a container object

Each container object is key'd by it's name and nested within "containers" of the top level structure above. A typical container object has the following structure

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

### Port object

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

### Volume object

```
{
  "description": "<A detailed description. Eg: This is where all incoming syncthing data will be stored>",
  "label": "<A short label. eg: Data Storage>",
  (optional)"min_size": <integer: suggested minimum size of the Share, in KB>
}
```

### Options object

An options object is a list of exactly two elements. (This needs to be improved or deprecated in favor of more specific design.)

`--net=host` would be represented as:

```
["--net", "host"]
```

Note that the opts field is a 2-d array, so the complete line for the above example looks like

```
"opts": [ ["--net", "host"] ],
```

### Command arguments object

A command arguments object is a list of exactly two elements detailing specific arguments to be passed onto the `docker run` command. As these arguments will simply be appended to the `docker run` command, they need to follow the same syntax and order. For instance, 

`docker run <...> image/name argument1 argument2="text2"` would be represented as:

```
["argument1", "argument2="text2"]
```

Note that, as for the options object, the cmd_arguments field is a 2-d array, so the complete line for the above example looks like

```
"cmd_arguments": [ ["argument1", "argument2="text2"] ],
```

### Environment object

```
{
  "description": "<Detailed description. Eg: Login username for Syncthing UI>",
  "label": "Web-UI username",
  (optional)"index": <integer: 1 or above. order of this environment variable, if relevant>
}
```


### Devices object

This optional object allows to pass a specific device to the Rock-on, and similarly to the Environment object, must have a "description" and "label".  
```
{
  "description": "<Detailed description of the device and its intent or specificities. Eg: path to device (/dev/xxx)>",
  "label": "Hardware encoding device",
  (optional)"index": <integer: 1 or above. order of this environment variable, if relevant>
}
```
Note that for the user, filling the fields corresponding to this object during the Rock-on installation is optional, allowing the users to leave the fields blank if not applicable to them.  
