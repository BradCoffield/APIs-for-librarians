---
layout: implementations
title: "This Week's Hours"
---
## Description
        
This will pull your library's hours for the current week from your LibCal system and display them. It shows the actual date for each listing so it's completely clear. Also, it highlights the hours for today's date.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

       
### More details
The API call, which is written in jQuery, will send your institution ID to Springshare and they will return JSON data with that week's open hours based on what you have setup inside LibCal.
        
Our code finds the appropriate hours and pulls them out, makes them more presentable, and displays them as part of an ```<ul>```. 
 
    
## The Code

#### HTML/CSS

{% highlight html linenos %}
<div id="weekly-hours-header">Library Hours:</div>
<ul id="this-weeks-hours"></ul>
{% endhighlight %}

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


{% highlight javascript linenos %}
 $(document).ready(function () {
    $.getJSON("https://api3.libcal.com/api_hours_grid.php?iid=000&format=json&weeks=1&systemTime=0&callback=?", function (json) {


        //takes the provided date and trims the year off the front   
    var sundayDate = json.locations[0].weeks[0].Sunday.date;
    var sundayShortDate = sundayDate.substr(5);
    var mondayDate = json.locations[0].weeks[0].Monday.date;
    var mondayShortDate = mondayDate.substr(5);
    var tuesdayDate = json.locations[0].weeks[0].Tuesday.date;
    var tuesdayShortDate = tuesdayDate.substr(5);
    var wednesdayDate = json.locations[0].weeks[0].Wednesday.date;
    var wednesdayShortDate = wednesdayDate.substr(5);
    var thursdayDate = json.locations[0].weeks[0].Thursday.date;
    var thursdayShortDate = thursdayDate.substr(5);
    var fridayDate = json.locations[0].weeks[0].Friday.date;
    var fridayShortDate = fridayDate.substr(5);
    var saturdayDate = json.locations[0].weeks[0].Saturday.date;
    var saturdayShortDate = saturdayDate.substr(5);


        //this works to remove front 0 from single digit months
    if (sundayDate[5] === '0') {
        sundayShortDate = sundayDate.substr(6)
    };
    if (mondayDate[5] === '0') {
        mondayShortDate = mondayDate.substr(6)
    };
    if (tuesdayDate[5] === '0') {
        tuesdayShortDate = tuesdayDate.substr(6)
    };
    if (wednesdayDate[5] === '0') {
        wednesdayShortDate = wednesdayDate.substr(6)
    };
    if (thursdayDate[5] === '0') {
        thursdayShortDate = thursdayDate.substr(6)
    };
    if (fridayDate[5] === '0') {
        fridayShortDate = fridayDate.substr(6)
    };
    if (saturdayDate[5] === '0') {
        saturdayShortDate = saturdayDate.substr(6)
    };

        //this takes the above and puts it with the name of the weekday and displays it in html

    $('#this-weeks-hours').append("<li id='sundays-hours'>Sunday, " + sundayShortDate + ": " + json.locations[0].weeks[0].Sunday.rendered + "</li");

    $('#this-weeks-hours').append("<li id='mondays-hours'>Monday, " + mondayShortDate + ": " + json.locations[0].weeks[0].Monday.rendered + "</li");

    $('#this-weeks-hours').append("<li id='tuesdays-hours'>Tuesday, " + tuesdayShortDate + ": " + json.locations[0].weeks[0].Tuesday.rendered + "</li");

    $('#this-weeks-hours').append("<li id='wednesdays-hours'>Wednesday, " + wednesdayShortDate + ": " + json.locations[0].weeks[0].Wednesday.rendered + "</li");

    $('#this-weeks-hours').append("<li id='thursdays-hours'>Thursday, " + thursdayShortDate + ": " + json.locations[0].weeks[0].Thursday.rendered + "</li");

    $('#this-weeks-hours').append("<li id='fridays-hours'>Friday, " + fridayShortDate + ": " + json.locations[0].weeks[0].Friday.rendered + "</li>");

    $('#this-weeks-hours').append("<li id='saturdays-hours'>Saturday, " + saturdayShortDate + ": " + json.locations[0].weeks[0].Saturday.rendered + "</li");


        //if you would like the hours for today marked in some way use the below and then style with .its-today
    var d = new Date();
    var n = d.getDay();
    switch (n) {
        case 0:
            $('#sundays-hours').addClass('its-today');
            break;
        case 1:
            $('#mondays-hours').addClass('its-today');
            break;
        case 2:
            $('#tuesdays-hours').addClass('its-today');
            break;
        case 3:
            $('#wednesdays-hours').addClass('its-today');
            break;
        case 4:
            $('#thursdays-hours').addClass('its-today');
            break;
        case 5:
            $('#fridays-hours').addClass('its-today');
            break;
        case 6:
            $('#saturdays-hours').addClass('its-today');
            break;
        }
    });

});
});{% endhighlight %}

* Line 2: Your ```iid=000``` will have a different number.
* Line 2: ```https://api3.libcal.com/api_hours_today.php?iid=000&lid=0&format=json&systemTime=0&callback=?``` is the link you'll generate through your LibCal admin area. Note the added ```&callback=?```
* Lines 6 - 19: Springshare sends the date in YYYY-MM-DD format and this code trims the year off. 
* Lines 23-43: Detects and trims the leading 0 from single digit months. You could also, with JS, change the dash to a '/' if you wanted and trim the 0 from days that start with it.
* Lines 47-59: This is what takes all our info and presents it on our website.
* Lines 63-88: Optional. This code will detect what day of the week it is (on page load) and apply a class to that day so you can highlight it in some way.
