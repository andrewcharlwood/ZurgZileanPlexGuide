# ZurgZileanPlexGuide 

## Glossary
RD = [Real Debrid](https://real-debrid.com/)

DMM = [Debrid Media Manager](https://debridmediamanager.com/search) 

SAR = Sonarr

RAR = Radarr

---

## Requirements/Overview
### Debrid subscription
#### [Real Debrid](https://real-debrid.com/) (£3.40/month)

There are other Debrid services, but they don’t integrate as well with the apps below.  More importantly, as we’re targeting cached files, we want the service with the most active users (and therefore the greatest number of files cached).

#### [Torbox](https://torbox.app/) (£2.34/month)

I have not used this personally - but it’s well integrated with the apps we’re using. One to keep an eye on.

### Managing rclone/streaming from real debrid server


#### [Zurg](https://github.com/debridmediamanager/zurg-testing) (Docker) (Free)

Provides a local webDAV server for your RD library. Also allows things like torrent repairs/reloading torrents to ensure they're not removed from the server (cached files are removed if no one accesses them for ~14 days).

[Compose file here](zurg/docker-compose.yml), some variation on flags vs the repo default.
[Config file here](zurg/config.yml), there's some regex matching to discern movies vs shows, and 4k vs non-4k. We don't use these folders (I had experimented with pointing plex directly at the rclone mount, it wasn't a good idea), but I didn't see them as causing any harm so have left them as is.



### Indexers


#### [Zilean](https://github.com/iPromKnight/zilean) (Docker) (Free)

Zilean is a must-have for a real debrid setup. It scrapes information from DMM to keep a local elastic search instance with a list of all movies currently **_cached_** on RD. Meaning no download is required to the RD server, so requests for new media are available as quickly as your system can process it.

##### Zilean docker compose file

The one on the repo is out of date. My compose file is [here](zilean/docker-compose.yml), note the port clashes with Tautilli default so if you use both make sure to change one.

#### [Torrentio](https://torrentio.strem.fun/configure) (API Endpoint) (Free)

Torrentio provides a rate-limited, public endpoint to a torrent indexer database. Torrentio scrapes indexer sites directly (rather than Zilean which scrapes DMM). Can be filtered to exclude uncached content if desired.

#### Zilean/Torrentio integration with Prowlarr

Install zilean with compose file above. You can check the endpoint `192.168.1.1:8181/swagger/index.html` to confirm it's working.

You will need to have setup Prowlarr. In the local directory for Prowlarr you will need to create the files:

```
prowlarr/Definitions/Custom/zilean.yml
prowlarr/Definitions/Custom/torrentio.yml
```
Templates available [here](https://github.com/dreulavelle/Prowlarr-Indexers/tree/main/Custom) - only the location of Zilean needs changing (if not on same docker network.)



### Request management:


#### [Overseerr](https://github.com/sct/overseerr) (Preferred) (Docker) (Free)

You already use this, so you know how great it is.


#### [Petio](https://github.com/petio-team/petio) (Alternative) (Docker) (Free)

Does the job but the UI/UX is just worser than Overseerr. Only benefit is the filter setup, allowing you to point different types of requests at different -arrs (rather than just normal vs 4k with Overseerr)


### Download/torrent selection manager


#### [Sonarr](https://github.com/Sonarr/Sonarr) (Docker) (Free)


#### [Radarr](https://github.com/Radarr/Radarr) (Docker) (Free)

**For setups with Overseerr,** you will require 1 instance for non-4k media, and 1 instance for 4k media for both SAR & RAR (Total: 4). 

**For setups with Petio**, you have greater flexibility in terms of being able to add a third instance for Anime, and a fourth for remux (Or as many instances as you have Petios filters) for both SAR& RAR (Total: 8+)



#### [Prowlarr ](https://github.com/Prowlarr/Prowlarr)(Docker) (Free)

Prowlarr pulls all the different indexers/sources for torrent sites into one location, and then keeps all your -arr instances in sync. We can use it to request from Zilean/Torrentio.


### “Download” manager

In this context, nothing is being downloaded locally. From the perspective of SAR/RAR, the “Downloader” acts in the same way a torrent client would. The “Download” manager takes the magnet file provided SAR/RAR, 


#### [Blackhole](https://github.com/westsurname/scripts) (Docker) (Free)

Similar to a torrent “blackhole”, where you have a watch folder where you can dump torrent files and they’re automatically collected, downloaded etc. In this case, magnet links/files are dumped into a folder, collected by the Blackhole script, and added to your RD account. Blackhole then monitors your local rclone mount for the files to appear, and creates a symlink to the “completed” folder, making it look like a normal download.


#### RDT–Client (Alternative) (Docker) (Free)

Just use blackhole it's much faster/better.

#### [Autoscan](https://github.com/cloudbox/autoscan) (Docker) (Free)

Replaces plex's scanning mechanism. When you import tens of thousands of items, the scanner will ruin your day/corrupt your plex database. Use autoscan. Example of my [config here](autoscan/config.yml).

---

## Guide


### [A Sailarr’s Guide to Plex + Real-Debrid](https://savvyguides.wiki/sailarrsguide/#prowlarr-torrentio)

Follow the above guide for the most part, with notes written above under Requirements/Overview.

---

# Riven

There are some references to Riven, I wouldn't recommend it right now as it's in very early development stages. It aims to be Overseer + -arrs + downloader + autoscan all in one. Currently it's still a bit buggy, but certainly something to keep an eye on. I use it for non-requests (like tracking MDB lists).
