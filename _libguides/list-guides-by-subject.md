---
layout: implementations
title: 'List Guides by Subject'
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
---
## Description
        
The LibGuides API allows you to limit the API's response to return only guides of a certain subject. Subjects are defined inside of LG, as assigned by admins or guide authors to individual guides.
        
A nice implementation of this idea exists at [Rochester Institute of Technology's LG homepage](http://infoguides.rit.edu/). Their page is what inspired my trying to use the API in this way.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

       
### More details
        
The API call, which is written in jQuery, will send your authorization key along with some pieces of information that will tell LG to only return JSON data about guides from a single subject or subjects.
        
Once that data is returned we pull out the individual title of a guide and its URL and place those pieces into an LI element. The jQuery will then enumerate through each JSON entry doing the same thing again and again until complete. You might have a list of 3 items or 300, it doesn't matter.
        
You link up the jQuery and your actual HTML using a specific ID name in your HTML, assigned to a UL. If you wanted a variety of different such lists you would need to use that many different pieces of this code, with your subject identifiers in the JSON call and the HTML/JS linkup changed.
    
        

## The Code

#### HTML/CSS

{% highlight html linenos %}
<ul id="business-guides"></ul>
{% endhighlight %}

#### JavaScript/jQuery


{% highlight javascript linenos %}

$.getJSON('https://lgapi-us.libapps.com/1.1/guides?site_id=foo1&key=foo2&status=1&subject_ids=38607', function (result) {
  var businessGuides = $.map(result, function (guidesData, index) {
    var listItem = $('<li></li>');
    $('<a href=/"' + guidesData.url + '/">' + guidesData.name + '</a>').appendTo(listItem);
    return listItem;
  });
  $('#business-guides').html(businessGuides);
});
{% endhighlight %}

#### Comments on the code:

* ```
foo1
```and ```
foo2
```are placeholders. Fill these in with the information provided by Springshare.
* ```
status=1
``` refers to published guides.
* ```
subject_ids=38607
``` refers to the specific code for the subject 'business' in our system. You'll need to replace that number.
* The ```
#business-guides
``` is what links your JS to your HTML. The ID here needs to match the ID of the UL in your code.