# Urbit and the Great Web 2.0 Tradeoff
> You can have strongly-owned identity, or you can have rich, seamless computing around identity, but you cannot have both.

## Intro: Young and Naïve
"I want a place to talk with my friends, share short text updates, make blog posts that the whole world can see, and collaborate on documents."

Sweet, it's 2020--you have tons of choices for this. Let's get cracking!

## Talking to My Friends
Let's do the easiest one first--a place to talk with your friends.  You message them all on Facebook and make a group and start sending messages. This is all fine until one of your members sees messages have been deleted because there was something insensitive.

"*facepalm*, of course we should have known to use a platform that reads our messages and also can ban users." You know Slack would be a bad choice for the same reason. You could do Telegram ("they seem ok about this, right?") but eventually you settle on Signal for maximum privacy, and your whole group chats away happily forever after (although not when they're on their computers instead of phones, but hey, we can't expect technology to seamlessy make life easier, right?)

## Share Short Text Updates
Alright, this one is super-easy. Twitter is the gold standard here: you can throw your messages out to the world and reply to anybody, and you can even make groups of trusted people and send DMs to other interesting accounts. I love Web 2.0.

Of course, you have to be a bit careful about what you say, because the wrong thing could get you reported and banned, and you'll lose all posts you've made and the social graph you've built up. No biggie, just have to lightly monitor every thought you have and also anticipate where Wokeness will be at 2 years from now so that you don't lose the asset then. About 30% of the mental energy for each Tweet goes to paying this anticipation/filtering tax.

You also should probably use an anon account so that mildly spicey but not bannable on Twitter things aren't reported to your employer, but this mental tax is definitely worth it.

You create a group that includes your Signal buddies and some new friends you make on here. Things are mostly good, although you're right back in the Facebook situation where you have to be careful about everything you say so that none of you lose your accounts.

## Porting Twitter Convos to Other Media
You're a savvy sophisticated tech user, and you're very aware that all the above mental taxes are arising because you're a guest on Twitter's platform and have to respect their rules. So you decide to [port your followers over to a new platform](https://github.com/balajis/twitter-export), or at least have alternate lines of contact to them.

You approach this in two ways:
1. You DM all the followers you like, telling them to follow you on X other platform. You could send them your phone number so that you can make Signal groups with them, but that's a bit weird.  You do that only for your Twitter besties. You send everyone else a link to your blog (see below).
2. You spam out messages on Twitter about the location of your new platform and put it in your profile. 

## Blogging
Now it's time to blog, both because you like the longer format, and because finally it's a piece of digital property you own.  You're not retarded, so you don't use Blogspot or Medium--yeah they handle all the sysadmin and hosting and SEO for you, but there's no point in doing a blog if it can be yanked away for violating XYZ Woke Talking Point that you forgot about.

So now you're into the wonderful world of server maintenance. This shouldn't be sooooo bad...just follow some tutorials online and get your Wordpress blog up and....OK I lied, it is that bad. If you're lucky you can cargo-cult your way through some commands on Digital Ocean, but otherwise you pay a few hundred bucks for someone to set it up for you.

Oh, and you didn't even start installing plugins yet.

But anyway you perservere because you're a normie American and you have that Protestant Work Ethic and your blog is finally up.  Some of your Twitter readers find it and start to comment regularly.

Consistent good commenting over time is a very-hard-to-fake proof-of-work, and so you decide to hold an irl bacchanalia for your favorite commenters. You require email signups, but a lot of your best commenters used throwaways to sign up. You make a post telling them to check those throwaways for a special announcement.  You also want to make a group to discuss the event, so you get them all in an email ring.

Did I mention that technology is great and reduces friction? Nothing manual or annoying here!

As you hang out in Los Angeles with your favorite commenters over some drinks talking about how much it's opened up your consciousness to go poly with other rationalist asexuals, you ask some of the more technically inclined how you can back up your posts on the blog. They laugh, and DB admin fun ensues.

## Collaborating on Documents
You find a way to message anyone you want to work with privately, get their Google id, and invite them to a Google doc. 

## Equilibrium
Eventually you get cancelled on Twitter because you don't participate in the 2022 Auto-Emasculation Challenge and thus are clearly transphobic. Your blog chugs along, and you still have your friends on Signal. You lose a Google document you had worked on for weeks because something in Google's black-box algo pattern-matched it as adjacent to some COVID-19 conspiracy theory. But the rest of your documents are kind of still there, so...winning?

## The Web 2.0 Tradeoff
In light of this highly representative story, we now can re-visit the Web 2.0 Tradeoff.

> You can have strongly-owned identity, or you can have rich, seamless computing around identity, but you cannot have both.

This tradeoff arises because in the current paradigm, a strong identity always has to be re-integrated into a given computing experience using authentication software.  The alternative is for the identity to become a line in a central company's database, which allows for arbitrarily rich experiences (unified ID across all Google apps, Twitter groups for known and loved Twitter friends, etc), but in return de-assetizes the ID--it's just a line in a company's DB at the end of the day.

We saw in our blog example what happens if you tried to handle this by using existing ID primitives like email or phone to run your own software and authentication: the rich, seamless computing experience is limited by the ability of you and all your users to become Unix sysadmins.

## There Is...Another Way
Now let's look at Urbit.

You boot up a ship, start using. Maybe join some groups.

Then one day, you use a bad word in one of the groups you run. Someone gets mad.  You block them.

And...that's it. You still have all your old contacts.  You still control all your data.

Now 2 weeks later a microblogging app comes out. It's able to import all the contacts you have from other groups and follow them automatically.

You start blogging, and any commenters you like get added by you to a group you make. Seamlessly.

Your social life, rather than experiencing friction, starts to *compound*. This is the promise of big Web 2.0 platforms, but across all your computing experience, with no "woke tax" constantly in the background of your mind.

## Urbit's Value Prop and Solution
Urbit seals off the computing layer and redefines it so that identity is a first-class primitive and the VM is standardized. Individually these two are necessary but not sufficient; together they are enough to break through the Web 2.0 Tradeoff.

Urbit solves the problem by making your identity be a line in a database, strengthening the guarantees about what that means. The database on Ethereum (very hard to cancel, and the identity implementation could be ported to another blockchain). The routing to you is handled by your star, and you can switch stars if your star tries to cancel.

For computing experiences, when you create an Urbit identity, it integrates seamlessly into any programs that are pre-installed or that you might install. In the example above, any cool blog commenters or Twitter followers could be seamlessly invited to the equivalent of a Slack or Signal group with a click of the button. It's such a change from the current paradigm that it feels like cheating.

Because Urbit works well for small groups already, it doesn't have the network effect bootstrapping problems you generally see in social apps. It works already, and people use it. If you want to be one of those people, come on down. If you're not ready yet, we'll keep making it easier to join until you are.

And they all lived happily ever after, the end.
