# 2018-05-09

## Project Outline

Assuming that we have to handle creating the kiosk web page ourselves, what will we require?

First, we have to decide between *push* and *pull* updates to the kiosk processors. The rest of this section assumes that we have chosen to *pull* updates from the processor.

* We can make changes to a single HTML file that is being served to all the signboards.
* We then just have to tell the browser running on the signboard to refresh its page from time to time.
* No login is required so as to get permission to store a file locally on the signboard processor.
* It is easier to add new signboards, since the server doesn't have to keep track of them.

> The alternative would require a service to be running on the signboard processors that would listen for messages from the central server containing updates to the page to display. This has advantages too: no server is required; individual signs can show different displays. Maybe others as well.
But it requires an app of some kind for updating the signboard processors, as well as the service app that runs on the processors themselves, so there is probably more code involved. Also it has to push to *all* signboard processors, meaning it has to be able to know how to find each of them.

So that means that we need a dedicated **server machine**. 
That is, a machine with fixed address (or able to broadcast a recognizable name) that is more or less always on.

The server machine has to be running a **web server**. 
It could be a proper web server app, like Nginx, or it could just be a web service that responds to HTTP GET requests from signboard processors. 

The server has to be backed by a **service** that constructs an appropriate HTML page. This might involve:

* calling the Google Calendar API and parsing the result for displayable _upcoming events_
* keeping track of a list of _announcement fliers_ to display in a rotating slideshow

The assumption is that **announcement fliers** are images, but there's no particular need for that to be so, as long as they are at least well-formed self-contained HTML (like a complete `div` element, for example).

The **current schedule** should come from Google Calendar, most likely. 
The assumption here is that there is no hand curation, apart from maybe specifying the number of upcoming events to display or something like that.

* There may have to be some rules about how the calendar is structured, in case we want to filter out certain events.
* Someday we might want to migrate the events to some other kind of database that we would query some other way for the signboards.

# 2018-05-10

Looking at Alice's experimental slides, and thinking about what she is already doing for *This week at Calvary* emails, trying to pull the schedule off of Google Calendar automatically seems like it might be overkill.

* Upload a tab-separated list of events in a single text file?
* Enter events into a web form?

Either way, it's clear that we are going to need some sort of *admin site*.


# 2018-05-21

## Maybe a Flask Server?

Looks like a fairly straightforward python web server. 

So a minimal parts list is starting to look like:

* Server machine
* Flask (and Python)
* Post _events_ and _adverts_ on an "admin" channel
* Get the bulletin board page on a "kiosk" channel

It is conceivable that we could have the kiosks talk to a service that is separate from the admin interface. The latter needs to be a web app, so that humans can post stuff. But the former could be a simpler web service. 

* The app updates the content data
* The service uses the latest data to construct a web page for the kiosks

## Peer Services

The key problem is how the kiosks should stay up to date with the server. 
It's not super important, but I bet Alice would want to know how things look as soon as she has posted new content. 
(Of course, she could do that from a browser, in general terms, but if there are geometry issues or scaling issues, she might want to see the real thing.)

It's not that hard to get the kiosks to periodically reload their page. But that does disrupt the flow of any slide shows, so you'd prefer if that only happened when they knew there was updated content. 

In order for the content server to push changes or notifications to the kiosks, there has to be a **peer service** running on the kiosk. 

1. The content server sends a notification to the kiosks
2. The kiosks send HTTP requests for the bulletin board page

And of course the content server as to know what kiosks exist (that is, their IP addresses). So as each new kiosk comes online, it has to *register* with the content server. 

And if the content server goes down, all the kiosks have to *re-register* once it comes up. Meaning there has to be some sort of heartbeat protocol, so that the kiosks can tell when they might need to re-register (i.e., on state change from "server lost" to "server available"). 

All of which sounds a bit complex.

