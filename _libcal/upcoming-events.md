---
layout: implementations
title: "Upcoming Events"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
tags:
- javascript
- jquery
- requires-springshare-auth-server
- client-side
---

## Description

This will display a number of upcoming events based on what you have setup in your LibCal. You can display the very next one (maybe with a heading of "Next Event") or everything for the next month (or year or whatever). The listed event(s) will be links that will allow users to register for the event. We use this to help promote our library workshops.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}
![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot2.jpg){:.screenshot}

### More details

{:.auth-server-promo}
Works with: [APIs for Librarians: Springshare Auth Server]({{site.baseurl}}/servers/springshare-auth-server)

The API call, which is written in jQuery, calls to our custom proxy server which gets authorized and then calls the LibCal API. Our code works with a single specific calendar (that you specify). If you want to have a master list of all events from your library you would need to create a single calendar in LibCal with them all. As mentioned above, you can change how many items from that calendar are displayed.

You will need your calendar's ID. To find this go, in the LibCal orange bar, to Calendars and then you'll see the ID in the left hand column of that table.

## The Code

#### HTML

{:.code-notes}

* The only essential part is `<ul id="upcoming-events-list"></ul>`. You can label it however you want.

{% highlight html linenos %}<div id="upcoming-events-container">
<h4 id="upcoming-events-header">Next Library Event</h4>
<ul id="upcoming-events-list"></ul>
</div>
{% endhighlight %}

#### CSS

{:.code-notes}

* Reminder: these styles are really just to make my screenshot look nice. You can use these or do whatever you want!

{% highlight css linenos %}
#upcoming-events-container {font-family: 'Martel', serif;font-size: 1.2em;}
#upcoming-events-container ul {list-style: none;}
.upcoming-events-time {font-size: .8em;margin: 0;}
#upcoming-events-header {margin: 0;}
{% endhighlight %}
 
#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}

* A lot of this code is to help make working with dates easy. Because working with dates in javascript = ugh.
* Line : `/events?cal_id=538` is where you put your calendar ID.
* Line 4: `YOUR_SERVER_HERE.herokuapp.com` will be replaced with your Heroku server's URL or whatever proxy server you're using to handle authentication. It's free and easy to use the one I built for you: [APIs for Librarians: Springshare Auth Server]({{site.baseurl}}/servers/springshare-auth-server)

##### The code itself:

{% highlight javascript linenos %} 
var apis4librarians_nextEvents = (function() {
  /* 
   * "f" is for Format & WHAT THE diff?? v0.5.0
   * Copyright (c) 2009 Joshua Faulkenberry
   * Dual licensed under the MIT and GPL licenses.
   */
  window.Date.prototype.f = function(format) {
    if (format == "@") {
      return this.getTime();
    } else {
      if (format == "REL") {
        var diff = (new Date().getTime() - this.getTime()) / 1000,
          day_diff = Math.floor(diff / 86400);
        return (
          (day_diff == 0 &&
            ((diff > -60 && "right now") ||
              (diff > -120 && "1 minute from now") ||
              (diff > -3600 && -Math.floor(diff / 60) + " minutes from now") ||
              (diff > -7200 && "1 hour ago") ||
              (diff > -86400 && -Math.floor(diff / 3600) + " hours from now") ||
              (diff < 60 && "just now") ||
              (diff < 120 && "1 minute ago") ||
              (diff < 3600 && Math.floor(diff / 60) + " minutes ago") ||
              (diff < 7200 && "1 hour ago") ||
              (diff < 86400 && Math.floor(diff / 3600) + " hours ago"))) ||
          (day_diff == 0 && "Tomorrow") ||
          (day_diff > -7 && -day_diff + " days from now") ||
          (-Math.ceil(day_diff / 7) == 1 && "1 week from now") ||
          (day_diff > -78 && -Math.ceil(day_diff / 7) + " weeks from now") ||
          (day_diff > -730 && -Math.ceil(day_diff / 30) + " months from now") ||
          (day_diff <= -730 && -Math.ceil(day_diff / 365) + " years from now") ||
          (day_diff == 1 && "Yesterday") ||
          (day_diff < 7 && day_diff + " days ago") ||
          (Math.ceil(day_diff / 7) == 1 && "1 week ago") ||
          (day_diff < 78 && Math.ceil(day_diff / 7) + " weeks ago") ||
          (day_diff < 730 && Math.ceil(day_diff / 30) + " months ago") ||
          Math.ceil(day_diff / 365) + " years ago"
        );
      }
    }
    var MONTH_NAMES = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December"
      ],
      DAY_NAMES = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
      LZ = function(x) {
        return (x < 0 || x > 9 ? "" : "0") + x;
      },
      date = this,
      format = format + "",
      result = "",
      i_format = 0,
      c = "",
      token = "",
      y = date.getYear() + "",
      M = date.getMonth() + 1,
      d = date.getDate(),
      E = date.getDay(),
      H = date.getHours(),
      m = date.getMinutes(),
      s = date.getSeconds(),
      yyyy,
      yy,
      MMM,
      MM,
      dd,
      hh,
      h,
      mm,
      ss,
      ampm,
      HH,
      H,
      KK,
      K,
      kk,
      k,
      value = new Object();
    if (y.length < 4) {
      y = "" + (y - 0 + 1900);
    }
    value.y = "" + y;
    value.yyyy = y;
    value.yy = y.substr(2, 4);
    value.M = M;
    value.MM = LZ(M);
    value.MMM = MONTH_NAMES[M - 1];
    value.NNN = MONTH_NAMES[M - 1].substr(0, 3);
    value.N = MONTH_NAMES[M - 1].substr(0, 1);
    value.d = d;
    value.dd = LZ(d);
    value.e = DAY_NAMES[E].substr(0, 1);
    value.ee = DAY_NAMES[E].substr(0, 2);
    value.E = DAY_NAMES[E].substr(0, 3);
    value.EE = DAY_NAMES[E];
    value.H = H;
    value.HH = LZ(H);
    if (H == 0) {
      value.h = 12;
    } else {
      if (H > 12) {
        value.h = H - 12;
      } else {
        value.h = H;
      }
    }
    value.hh = LZ(value.h);
    if (H > 11) {
      value.K = H - 12;
    } else {
      value.K = H;
    }
    value.k = H + 1;
    value.KK = LZ(value.K);
    value.kk = LZ(value.k);
    if (H > 11) {
      value.a = "PM";
    } else {
      value.a = "AM";
    }
    value.m = m;
    value.mm = LZ(m);
    value.s = s;
    value.ss = LZ(s);
    while (i_format < format.length) {
      c = format.charAt(i_format);
      token = "";
      while (format.charAt(i_format) == c && i_format < format.length) {
        token += format.charAt(i_format++);
      }
      if (value[token] != null) {
        result = result + value[token];
      } else {
        result = result + token;
      }
    }
    return result;
  };
  $.getJSON(
    "https://pre-publish-springshare.herokuapp.com/springshare/libcal/passthrough?what=/events?cal_id=538",
    function(result) {
      var howManyToDisplay = 100;
      if (howManyToDisplay > result.events.length) {
        howManyToDisplay = result.events.length;
      }
      console.log(result);

      let displayUpcomingEvents = (function(howManyToDisplay) {
        var eventsUL = document.getElementById("upcoming-events-list");

        let startDateTime = new Date(result.events[0].start);
        console.log(startDateTime, startDateTime.valueOf());

        for (i = 0; i < howManyToDisplay; i++) {
          let eventsFun = result.events[i];
          let startTime = new Date(eventsFun.start);
          let formattedStartTime = startTime.f("hh:mma EE, MMM d");
          console.log(startTime);

          eventsUL.insertAdjacentHTML(
            "beforeend",
            `<li id="event-${eventsFun.id}"><a href="${eventsFun.url.public}">${
              eventsFun.title
            }</a> <ul><li class="upcoming-events-time">${formattedStartTime}</li></ul> </li>`
          );
          console.log(result.events[i].title);
        }
      })(howManyToDisplay);

      //id title start end url location.name remaining seats=.seats - .seats_taken
    }
  );
})();


{% endhighlight %}
