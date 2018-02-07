---
layout: implementations
title: "Springshare Auth Server"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
tags:
- javascript
- node
- server
- express
- heroku
- springshare
---

## What's the deal with this?

The new v1.2 API endpoints for LibGuides and LibCal require* that you do not expose your private authorization details to the world. If you put them in code in a widget on a libguide they would be exposed (naughty auth details!). So, the solution to this is to have them living on a webserver where no one else can see them.

I've published the code (on [GitHub](https://github.com/BradCoffield/auth-server-for-springshare)) for a webserver written in Node and Express that will hide your auth details, and when used properly will communicate with the LC/LG APIs and return the information you desire.

*(highly highly suggest)
 
## Deploying to Heroku

You can use my code however you like, of course. But it's very easy to get this up and hosted on Heroku for free. [Heroku](https://www.heroku.com) is a well-regarded host that plays nicely with Node.js and other great technologies. (FYI: I'm not affiliated with Heroku in any way and can assume no liability in your using them blah blah blah.)

To create your own webserver on Heroku with my code: 

{:.small-ul}
* Go to [Heroku](https://www.heroku.com) and create an account.
* Come back here and click the 'Deploy to Heroku' button: [![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/BradCoffield/auth-server-for-springshare)
* You will go to a new page where you can create a name for your server. You will also need to add your authentication information for both LibCal and LibGuides. You can find this information on their respective API pages under the Authentication tab.

## Things to consider with Heroku

The free tier of Heroku is great. The only downside is that your server will sleep after 30 minutes of no use. After sleeping it takes ~20 seconds for it to spin back up and respond to that waking request. There are services that will periodically ping your server to keep it awake, which helps remediate this problem. Also keep in mind that free tier apps (our server) have to sleep at least 6 hours per day. Check out [Kaffeine](http://kaffeine.herokuapp.com/#!) which seems to take that into account and won't ping 24 hours per day.

The next pricing tier up is only 7 dollars per month if you don't want to worry about the sleeping issue.

## Okay so I have it deployed, now what?

Well, now you can make your requests for data from the respective Springshare APIs to your (new, shiny, and extra special) server.

If you wanted a list of your AZ assets from LibGuides the documentation says you should query the url: `https://lgapi-us.libapps.com/1.2/az`. Instead you would query your new server like this: `https://www.your-new-server.herokuapp.com/springshare/libguides/passthrough?what="/az"`

In LibCal, for a list of spaces by category they say you should query `https://api2.libcal.com/1.1/space/categories/:id`. Instead you would query your server like this: `https://www.your-new-server.herokuapp.com/libcal/passthrough?what="/space/categories/3162"`

**What's going on here?** Basically you pull the last part(s) of the valid query you would make to them and put it in quotes there after `?what=`. Do include the forward slash at the start of what's in quotes.

## Even simpler!

Check out these implementations that work with your custom auth server!

{:.small-ul}
* [LibGuides: Suggest Random Database]({{site.baseurl}}/libguides/random-database)
* [LibCal: Room Availability]({{site.baseurl}}/libcal/room-availability)
* [LibCal: Upcoming Events]({{site.baseurl}}/libcal/upcoming-events)

