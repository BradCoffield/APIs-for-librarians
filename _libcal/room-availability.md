---
layout: implementations
title: "Room Availability"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
tags:
- javascript
- jquery
- requires-springshare-auth-server
- client-side
---

## Description

LibCal allows you to set up "Spaces" and this code checks the spaces you've created and will alert users if they are currently available (or not). This would be great for private study rooms, makerspaces, or even limited-access special collections.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

### More details

{:.auth-server-promo}
Works with: [APIs for Librarians: Springshare Auth Server]({{site.baseurl}}/servers/springshare-auth-server)

The API call, which is written in jQuery, calls to our custom proxy server which gets authorized and then calls the LibCal API. You specify which category of spaces you want information on. To get the category ID go to Admin > Equipment &amp; Spaces > Click the link for spaces in the chart there > The ID is next to the category heading

I set this up to work with our private study rooms and they are booked first-come first-served basis and students can use them for one hour at a time. In LibCal our rooms are set to be available every five minutes and with a minimum booking time of 60 minutes. The javascript belows checks to see if the room is available within the next 10 minutes and if so says that the room is available. If you had a space that was booked say, every hour on the hour, you would need to tweak the code below to properly check if the space was available.

## The Code

#### HTML

{:.code-notes}

* The only truly necessary part is `<ul id="quiet-rooms-list"></ul>`. The rest is for styling and presentation purposes and you can do whatever you want.

{% highlight html linenos %}

<div id="quiet-rooms-container">
    <h4 id="quiet-rooms-header">Quiet Study Rooms</h4>
    <ul id="quiet-rooms-list"></ul>
</div>
{% endhighlight %}

#### CSS

{:.code-notes}

* This code uses font-awesome icons. These are already available in Springshare products but if you are using this on your library website or any other system you may need to load them yourself.
* Reminder: these styles are really just to make my screenshot look nice. You can use these or do whatever you want!

{% highlight css linenos %}
#quiet-rooms-container {font-family: 'Martel', serif;}
#quiet-rooms-list {list-style: none;margin-top: 3px;padding-left: 10px;}
.fa-check {color: green;margin-left: 3px;font-size: 1.1em;}
.fa-lock {color: #FF4136;margin-left: 3px;font-size: 1.1em;}
#quiet-rooms-header {margin: 0px;}
{% endhighlight %}

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}

* Line 3: `YOUR_SERVER_HERE.herokuapp.com` will be replaced with your Heroku server's URL or whatever proxy server you're using to handle authentication. It's free and easy to use the one I built for you: [APIs for Librarians: Springshare Auth Server]({{site.baseurl}}/servers/springshare-auth-server)
* Line 3: `/space/category/5366?availability` is what you can customize for your system. The `5366` is the category ID. (See above for instructions on finding the category id in your system).

##### The code itself:

{% highlight javascript linenos %}var apis4librarians_roomAvailable = (function() {
var getRoomAvailability = function() {
$.getJSON(
"https://YOUR_SERVER_HERE.herokuapp.com/springshare/libcal/passthrough?what=/space/category/5366?availability",
function(result) {
let roomsArr = result[0].items;
let now = new Date();
let soon = new Date(roomsArr[0].availability[1].from);

        var secondArr = roomsArr.map(room => {
          var roomsUL = document.getElementById("quiet-rooms-list");
          roomsUL.insertAdjacentHTML("beforeend", `<li id="${room.id}">${room.name} `);

          return {
            roomID: room.id,
            soonTime: room.availability[1].from /* [1] and not [0] because of a delay on their end. This is safer. */
          };
        });
        secondArr.map(pieces => {
          let piecesTime = new Date(pieces.soonTime);

          if (piecesTime - now <= 600000 /* 10 minutes in milliseconds */) {
            var individualRoom = document.getElementById(pieces.roomID);
            individualRoom.insertAdjacentHTML(
              "beforeend",
              `<i class="fa fa-check"  title="Room currently available." aria-label="Room currently available."></i>`
            );
          } else {
            var individualRoom = document.getElementById(pieces.roomID);
            individualRoom.insertAdjacentHTML(
              "beforeend",
              `<i class="fa fa-lock"  title="Room currently unavailable." aria-label="Room currently unavailable."></i>`
            );
          }
        });
      }
    );

};
getRoomAvailability();
})();

{% endhighlight %}
