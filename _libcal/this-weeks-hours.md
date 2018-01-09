---
layout: implementations
title: "This Week's Hours"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
---

## Description

This will pull your library's hours for the current week from your LibCal system and display them. It shows the actual date for each listing so it's completely clear. Also, it highlights the hours for today's date.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

### More details

The API call, which is written in jQuery, will send your institution ID to Springshare and they will return JSON data with that week's open hours based on what you have setup inside LibCal.

Our code finds the appropriate hours and pulls them out, makes them more presentable, and displays them as part of an `<ul>`.

## The Code

#### HTML

{% highlight html linenos %}

<div id="weekly-hours-header">Library Hours:</div>
<ul id="this-weeks-hours"></ul>
{% endhighlight %}

#### CSS

{% highlight css linenos %}
.its-today {
background-color: rgba(215, 119, 121, 0.1);
}

#this-weeks-hours {
list-style: none;
padding: 5px 0 0 5px;
}

.dates {
font-family: 'Martel', serif;
font-size: 1.2em;
padding: 0 .3em;
}

.day-of-week {
font-family: 'Martel', serif;
font-size: 1.2em;
font-weight: bold;
}

#weekly-hours-header {
font-family: 'Martel', serif;
font-size: 1.5em;
font-weight: bold;
}{% endhighlight %}

* You probably already have your fonts set. But Martel is used above and is available through Google Fonts.

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}

* You will need to go to your LibCal admin area. Admin > Hours > Widgets > Weekly Data. And then "Generate Code Preview." This will provide you with the information you need below.
* Line 3: Your `iid=000` will have a different number.
* Line 3: `https://api3.libcal.com/api_hours_today.php?iid=000&lid=0&format=json&systemTime=0&callback=?` is the link you'll generate through your LibCal admin area. Note the added `&callback=?`
* Lines 49-76: Optional. This code will detect what day of the week it is (on page load) and apply a class to that day so you can highlight it in some way.

##### The code itself:

{% highlight javascript linenos %}
varapis4librarians_WeeksHours = (function() {
  $.getJSON(
    "https://api3.libcal.com/api_hours_grid.php?iid=000&format=json&weeks=1&systemTime=0&callback=?",
    function(json) {
      class DaysHours {
        constructor(singleDay) {
          vartheDate;
          //process and clean up the input
          if (singleDay.date[5] === "0") {
            this.theDate = singleDay.date.substr(6);
          } else {
            this.theDate = singleDay.date.substr(5);
          }
          this.rendered = singleDay.rendered;
        }
        renderMe(dayName) {
          $("#this-weeks-hours").append(
            "<li id='" +
              dayName +
              "-hours'><span class='day-of-week'>" +
              dayName +
              ", " +
              this.theDate +
              ":</span> " +
              "<span class='dates'>" +
              this.rendered +
              "</li>"
          );
        }
      }

      const daysOfTheWeek = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday"
      ];
      const locationDays = json.locations[0].weeks[0];
      const daysRendered = {};

      for (varday of daysOfTheWeek) {
        vardayToRender = new DaysHours(locationDays[day]);
        varkey = `${day}_stuff`;
        daysRendered[key] = dayToRender.renderMe(day);
      }
      varhighlightDay = (function() {
        //if you would like the hours for today marked in some way use the below and then style with .its-today
        var d = new Date();
        var n = d.getDay();
        switch (n) {
          case 0:
            $("#Sunday-hours .dates").addClass("its-today");
            break;
          case 1:
            $("#Monday-hours .dates").addClass("its-today");
            break;
          case 2:
            $("#Tuesday-hours .dates").addClass("its-today");
            break;
          case 3:
            $("#Wednesday-hours .dates").addClass("its-today");
            break;
          case 4:
            $("#Thursday-hours .dates").addClass("its-today");
            break;
          case 5:
            $("#Friday-hours .dates").addClass("its-today");
            break;
          case 6:
            $("#Saturday-hours .dates").addClass("its-today");
            break;
        }
      })();
    }
  );
})();
{% endhighlight %}
