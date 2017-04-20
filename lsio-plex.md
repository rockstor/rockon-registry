Plex organizes video, music and photos from personal media libraries and streams them to smart TVs, streaming boxes and mobile devices.


**Parameters**

* `/config` - Plex library location. *This can grow very large, 50gb+ is likely for a large collection.*
* `/transcode` - Transcode directory to offload heavy writes in a docker container
* `/data` - Media goes here. Add as many as needed e.g. `/data/movies`, `/data/tv`, etc.
* `VERSION` - Set this to a full version number if you want to use a specific version e.g. `0.9.12.4.1192-9a47d21`, or set it to `plexpass` or `latest`
* `PGID` for for GroupID - see below for explanation
* `PUID` for for UserID - see below for explanation