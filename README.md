# rockon-registry

This repository consists of Rock-on(Docker based apss) configuration profiles
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
      "version": "<optional: arbitrary version string>",
      "website": "<Underlying app website>",
      (optional)"icon": "<link to icon, if any>",
      (optional)"more_info": "<string or html with more information to display to the user in the Rockstor UI>",
      (optional)"volume_add_support": <true, If the app allows arbitrary Shares to be mapped to the main container>,
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
  (optional)"launch_order": "1 or above. If there are multiple containers and they must be started in order, specify here.>",
  (optional)"ports": {
    "<container side port number1>": <port object represending a port mapping between host and container. see below>,
    "<port number2>": <another port object, if necessary. see below>, ...
  },
  (optional)"volumes": {
    "<path1 inside container>": <volume object representing a Share<->directory mapping in the container. see below>,
    "<path2 inside container>": <another volume object, if necessary. see below>, ...
  },
  (optional)"opts": [ An array of option objects that represent container options such as --net=host etc.. see below],
  (optional)"environment": {
    "<env var1 name>": <env object representing one environment variable required by this container. see below>,
    "<env var2 name>": <another env object, if necessary. see below>, ...
  }
}
```
As it is evident from above, a container object has nested objects for port and volume mappings, container options and environment variables. These are described below.

### Port object

```
{
  "description": "<A detailed description of this port mapping, why it's for etc..>",
  "label": "<A short label for this mapping. eg: Web-UI port>",
  "host_default": <integer: suggested port number on the host. eg: 8080>,
  (optional)"protocol": "<tcp or udp>"
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

An options object is a list of up to two elements. (This needs to be improved or deprecated in favor of more specific design.)

```
[ "<eg: --net=host>" ]
```

### Environment object

```
{
  "description": "<Detailed description. Eg: Login username for Syncthing UI>",
  "label": "Web-UI username",
  (optional)"index": <integer: 1 or above. order of this environment variable, if relevant>
}
```
