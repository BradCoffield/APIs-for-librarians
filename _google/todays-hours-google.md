---
layout: implementations
title: "Google Calendar 'Today's Hours' Plugin"
author: "Caryl Wyatt"
author_email: "caryl.e.wyatt@gmail.com"
tags:
- javascript
- jquery
- google-calendar
---
{:.implementation-author}
This page courtesy of: [{{page.author}}](http://carylwyatt.github.io/)

### Description

This plugin takes information from a single Google calendar and displays it in a table. [Ivy Tech Community College Indianapolis Library's website](http://library.ivytech.edu/indianapolis) (via Springshare's LibGuides CMS) uses this tool to display "Today's Hours" for three library locations on the homepage. 

### Screenshot
This is how it's displayed on our website, but you can style however you like.

![screenshot of today's hours](https://github.com/carylwyatt/google-cal-hours-widget/blob/master/todays-hours-screenshot.jpg?raw=true){:.screenshot}

### More details
You will need access to the settings of the Google Calendar you want to get data from. You will also need to access the [Google API Manager](https://console.developers.google.com/apis/). 

#### Calendar setup
This script reads specific data points from the calendar's JSON data. If you don't want to fiddle with the JS, you'll have to set up your calendar and events exactly like mine. Feel free to do things your own way, but you'll have to edit the code. 

#### Calendar settings
Access permissions: check **make available to public** box

#### Events

- Summary can be whatever will make it easy for you to read
- Description is what will actually show up on the calendar
	- if the description is left **blank**, the event won't show up at all
- If you want the open and close times to show up, make sure you enter them into the event
- If you are closed, don't enter a start/end time
	- instead check the "**all-day event**" box for the event
	- this will cause the script to list "**closed**" on the hours widget

## Code

### HTML

{:.code-notes}

- Lines 1-2 load the required JS libraries; this can go in the `head`, but if you're using LibGuides, just drop this whole HTML section into the Source section of a rich text box
- Lines 4-8 are necessary, the rest are window dressing

{% highlight html linenos %}
<!--lodash--><script src="https://cdn.jsdelivr.net/lodash/4.17.4/lodash.min.js"></script>
<!--moment--><script src="https://cdnjs.cloudflare.com/ajax/libs/moment.js/2.18.1/moment.min.js"></script>
    <h3>Today's Hours</h3>
    <p id="todays-date"></p>
    <div class="table-responsive">
      <table class="table table-condensed table-hover" id="hours-grid">
      </table>
    </div>
    <p><a class="more-link" href="http://library.ivytech.edu/indianapolis/hours">See all hours and locations  <i class="fa fa-arrow-right" aria-hidden="true"></i></a></p>
{% endhighlight %}

### CSS

{% highlight css linenos %}
    #todays-date {
        font-style:italic;
    }
    
    #hours-grid td {
        border-top: none;
    }
    
    #hours-grid .library-location {
        font-size: 18px;
        font-weight: 700;
        padding-left:0;
    }
    
    .table-responsive {
		border:none;
    }
{% endhighlight %}

### JavaScript/jQuery

#### Notes for implementation

{:.code-notes}

- Line 16: You'll find your calendar address in the settings of the google calendar. It looks like an email address and follows the phrase **Calendar ID**.
- Line 17: The API key is supplied by the Google Developers Console API Manager. You will need to create a new credential for this project, and this is free to do. See [Creating a Google API Key](https://docs.simplecalendar.io/google-api-key/) for step-by-step instructions.
- Line 21: The time frame is set to read your google calendar from 1 am - 11:59 pm. I had trouble with all-day events showing up on the next day when I used the 0:00:00-23:59:59 time code, so I tweaked it. Our library is never open past 7pm, so I've never tested a time frame that extends to the next day. YMMV.

{% highlight javascript linenos %}
  //====set up date/time variables (moments.js)
    //get today's date
    const date = moment();
    
    //format it so it's in plain english
    //e.g. Thursday, March 30, 2017
    const today = date.format("dddd, MMMM D, YYYY");
    
    //append today's date to empty div
    $('#todays-date').append(today);
    
    //create variable to hold utc-style of today's date (moments.js)
    const utc = date.format("YYYY-MM-DD");
    
    //create variables for calendar address and api key
    const calAddress = "xxxxxxxxxxxxxxxx@group.calendar.google.com";
    const keyAPI = "API KEY";
    
    //inject today's date in YYYY-MM-DD format to the API URL
    //this url returns only the events scheduled for today
    const googleCal = `https://www.googleapis.com/calendar/v3/calendars/${calAddress}/events?singleEvents=true&timeMin=${utc}T01:00:00-04:00&timeMax=${utc}T23:59:59-04:00&key=${keyAPI}`;
    
    //===========MAIN PLUGIN FUNCTION=============
    //takes response data from API (see const response below) and runs it
    const displaySchedule = (data) => {

	//returns an array that holds all calendar data for specific day (lodash.js)
    const libraries = _.filter(response.responseJSON.items);

    	//sorted by reverse alphabetical order (lodash.js)
      const sorted = _.orderBy(libraries, 'summary', 'desc');
    
    //and placed in a table
    let tableHTML = '<tbody>';
     
      //loops through each event in the sorted libraries array/google calendar events (lodash.js)
      _.forEach(sorted, function(library) {
        
        //if there is a dateTime key in the start and end of the calendar event
        if (library.start.dateTime && library.end.dateTime) {
        
            //takes given time string (ISO 8601), parses it to standard format, then returns formatted time (moments.js)
            //e.g. "2017-03-30T21:00:00-04:00" -> "9 pm"
            function timeChanger(time) {
              const getDate = moment.parseZone(time);
              const formatDate = getDate.format("h a");
              return formatDate;
            }
            
            //setting up variables to grab start and end times from the google calendar API
            //put those times through time changer function
            let startTime = timeChanger(library.start.dateTime);
            let endTime = timeChanger(library.end.dateTime);
           
  
            
              //creates table row for each event in the calendar array
              //e.g. | Downtown | 8 am - 9 pm |
             tableHTML += `<tr><td class="library-location">${library.description}</td><td>${startTime} - ${endTime}</td></tr>`;
          
  
        //if there is no description in the event, skip it
          //we occassionally have an employee add an event on accident, but they normally don't include a description; this skips that event
        } else if((library.start.dateTime && library.end.dateTime && !library.description) || !library.description) {
          
          return;
        
        //otherwise, it's closed! (dateTime returns undefined)
        } else {
          
              tableHTML += `<tr><td class="library-location">${library.description}</td><td>Closed</td></li>`;
        }
      }); //end of foreach loop
    
    tableHTML += '</tbody>';
    
    //empty old schedule tbody (may be unnecessary, but you never know) and replace with new tbody
    $('#hours-grid').empty();
    $('#hours-grid').append(tableHTML);
    } //end display schedule function
    
    //google calendar API request call
    const response = $.getJSON(googleCal, displaySchedule);
{% endhighlight %}