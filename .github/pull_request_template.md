> To ease and accelerate the PR submission, please use the template below to provide all requested information.
> You can find guidelines on which docker image to use in the [Rockstor's documentation](http://rockstor.com/docs/contribute_rockons.html#which-docker-image-should-i-use), 
> as well as examples of proper formatting in other rock-ons ([here](https://github.com/rockstor/rockon-registry/blob/master/teamspeak3.json)
> and [here](https://github.com/rockstor/rockon-registry/blob/master/nextcloud-official.json))



Fixes # .

### General information on project
This pull request proposes to add a new rock-on for the following project:
- name:
- website:
- description: <brief description of the project and how it intergrates with Rockstor>

### Information on docker image
- docker image: <link to image page on docker hub>
- is an official docker image available for this project?: 


### Checklist
- [ ] Passes [JSONlint](https://jsonlint.com) validation
- [ ] Entry added to `root.json` in _alphabetical_ order (for new rock-on only)
- [ ] `"description"` object lists and links to the docker image used
- [ ] `"description"` object provides information on the image's particularities (advantage over another existing rock-on for the same project, for instance)
- [ ] `"website"` object links to project's main website
