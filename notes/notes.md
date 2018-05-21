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

