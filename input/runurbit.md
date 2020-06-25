# Get an Urbit Ship on Your Mac and Do Fun Stuff

An Urbit "ship" is the Urbit program running with your id.

This will teach you how to get one up quickly. It requires a Mac (or Linux, but I'm assuming if you use Linux this is too noobish for you. Do the same instructions but use the Linux binary that I'll link below).

**Important**
Once you start your ship, **do not start another copy of it**. You will have problems if you try to run the same ship twice.  We will repeat this warning throughout the tutorial.

## I Got an Invite!
* You got an email saying "You are invited to Urbit"
* Click the link to accept the invite

### Activate
* On the page you end up at ("Activate"), click "Go". 
* This will generate a "passport" for you, which is what lets you own your identity
* Download the Passport
* It has two files:
  - Management Proxy
  - Master Ticket
* Save both somewhere safe, or print them. I save mine in Lastpass Secure Notes
* Hit "Continue" when done

### Use Your Master Ticket
On the next screen, input the four-word phrase from your Master Ticket. Wait while your planet is set up. This takes a few minutes.

## Start Your Planet Running
Open Terminal, and type (you can cut and paste these commands):
```
mkdir ~/urbit
cd ~/urbit
curl -O https://bootstrap.urbit.org/urbit-v0.10.6-darwin.tgz
tar xzf urbit-v0.10.6-darwin.tgz
cd urbit-v0.10.6-darwin.tgz
./urbit KEY STUFF
```

## Join a Chat
