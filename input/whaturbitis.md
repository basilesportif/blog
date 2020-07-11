# What Urbit Is
tl;dr Urbit wants to be *the* p2p server OS+database. It does not want to be a client OS. 

If you think of the current client+server space, Linux completely dominates servers, while Windows, Android, iOS, OS X, and Linux desktop compete for and coexist in the client space. Urbit is targeting the former market, not the latter.

A world in which Urbit wins looks a lot like our current one on the client side (barring further innovations).  However, on the server side, the huge cloud backends of a lot of social media and Saas companies are downsized substantially; they are necessary when all of a service's users live in one giant database. The huge array of cloud services in AWS etc would also be devalued since they also exist for that use case. Urbit only needs commoditized Unix server+storage configurations.

Because Urbit is so isolated and self-contained, it is already able to commoditize whatever hardware it runs on. I switched my ship's hosting from my laptop to AWS to Digital Ocean to Hetzner with ~0 glitches or abstraction leakages.

## Base System Description

### Overview
Urbit is a deterministic VM with an integrated ACID database. It allows encrypted networking with other Urbit VM's on a private overlay network where communication is peer-to-peer, identities are stable, and peer discovery is negotiated by galaxies (eventually will be done by stars).  The OS kernel provides a specific mechanism for userspace applications to upgrade themselves and transition their internal databases to new schema.

### Identity
The source of truth for identities is an Ethereum contract (program/db) called Azimuth, which can be interfaced with using direct calls to the Eth blockchain or, more conveniently, but making those calls through [bridge.urbit.org](https://bridge.urbit.org).

Programs inside Urbit make HTTP calls out to read the Ethereum blockchain when they need to verify an identity's owner.

In the future I personally think this program will move to a faster private blockchain run on galaxies, but it's fine for the time being.

### Updates to Kernel and Interpreter
The system's kernel can update itself using "OTA"s, through a process described [here](https://gist.github.com/belisarius222/e9e4b382eda75ad788addf317eda8d99). These updates can change the system from its lowest layers all the way through userspace applications.

The Urbit Unix interpreter also can be updated. These updates do not affect system behavior, since they don't change Nock itself. However, they can drastically improve performance and reliability in the same way that improved JS VM implementations made rich clientside webapps possible circa 2010.

## What Urbit Does Well
* p2p communication and discovery: if you have a program, and you want it only accessible to certain other identities, you literally can just store those identities in a set, and check for membership as your authentication.
* exactly-once messaging: this is enforced at the network protocol level, which means you can have an app mirror another ship's state and it "just works"
* remote data subscriptions: you can subscribe to declared local and remote data sources inside other apps with a simple command in a userspace app, and you will get typed, validatable data back over the wire.
* decouples static resources and code ("filesystem") from application dbs. This lets the two things be handled differently logically for updates
* OTA (live kernel upgrades) -- see [Ford Fusion writeup](https://gist.github.com/belisarius222/e9e4b382eda75ad788addf317eda8d99) and [Why Hoon](https://urbit.org/blog/why-hoon/) for why this is possible and how it works
* secure, programmable identity
* negotiate connections for high bandwith apps
  - binary data: allows seamless, integrated S3 storage
  - streaming: can just exchange links in another service
  - by way of (very loose) analogy, think of torrent tracker sites vs BitTorrent itself
* HTTP sync of typed data to/from client applications
  - read [this document](https://github.com/timlucmiptev/gall-guide/blob/master/guide-docs/chanel.md)'s "channel.js" section, as well as the `index.html` and `index.js` files (no Hoon required)
* call out to external HTTP resources is simple programmatically

## Constraints
* every event has to be processed and written to disk
* many of those trigger messages out to networked subscribers
* system isn’t great for holding binary data (although easy to decouple that and integrate it remotely)
* will never (imo) service low-latency, high-bandwidth cases like graphical client UIs, intensive game-processing, etc.

## Security Issues
* not audited; lots of obvious problems
* running on cloud hardware
* cryptographic functions have to be jetted to avoid certain attacks; adds surface area
* system data is accessible to someone who has access to the machine an Urbit is running on

## Performance Issues
The interpreter doesn't do anything smart to handle the fact that all this is running on a single thread. As a result, operations that should be more async can block each other. The easy example would be a ship that was both serving static, frequently accessed content to the world and also being used by its owner to participate in and serve chat groups. The two could block each other.

There are approaches in the pipeline for this and it's worth discussing futher in depth with the kernel/interpreter-level Tlon engineers.

## Things Urbit Desperately Needs
These are things that are pretty imo necessary conditions for widespread adoption, even among the technically enthusiastic and smart
### Userspace App Backup/Restore
To provide a sense of permanence, Urbit needs to decouple at least parts of the blob DB from the event log. The system is really well-designed to serialize state, and the Gall kernel vane (which manages userspace apps) *already* tracks app metadata, so this is very achievable. iirc, Tlon already (apparently?) has a working internal version of this, but would have to ask.

### 3rd Party App Install
This process is currently not easy, and also has the possibility to crash your ship or get it into a hard-to-recover-from inconsistent state. Tlon is aware of the issue and I hear there is stuff in internal development, but not 100% sure.

### Automated Hosting
Enthusiastic Urbiters are already hosting instances for friends and family in the cloud. This is ~100x easier for Urbit than for a normal Unix app because of the way that Urbit commoditizes cloud hardware, but still costs money. I've heard there is stuff in the pipeline from Tlon to do "push button, pay money, get hosting", but I'd have to see.
