---
layout: implementations
title: 'Display a Random LibGuide Link on Page Load'
---

## Description
        
This code will pull a random guide from your system's published guides and display it as a simple link. Also, there's a button that will regenerate the randomized guide.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

       
### More details
The API call, which is written in jQuery, will send your authorization key along with some pieces of information that will tell LG to only return JSON data about guides from a single subject or subjects.
        
Once that data is returned we pull out the title of the guide and also its URL and place those pieces into a link element. The jQuery will then enumerate through each JSON entry doing the same thing again and again until complete. You might have a list of 3 items or 300, it doesn't matter.
        
You link up the jQuery and your actual HTML using a specific ID name in your HTML, assigned to an ```<a>```. 

Pressing the button will re-run everything and display a new link.
 
    
## The Code

#### HTML/CSS

{% highlight html linenos %}
<div id="foo"></div>
<input type="button" id="btn" value="Click" />

{% endhighlight %}

* Line 1: This shows the link.
* Line 2: This generates the link which, when clicked, will re-roll the guide.
* No CSS in the example.

#### JavaScript/jQuery


{% highlight javascript linenos %}
$(document).ready(function(){
    $.getJSON('https://lgapi-us.libapps.com/1.1/guides?site_id=foo1&key=foo2&status=1', function (result) {
      var entry = result[Math.floor(Math.random() * result.length)];
      var randomGuide = '<a href="' + entry.url + '">' + entry.name + '</a>';
      $('#foo').html(randomGuide);
  });
    $("#btn").click(function(){
        $.getJSON('https://lgapi-us.libapps.com/1.1/guides?site_id=foo1&key=foo2&status=1', function (result) {
            var entry = result[Math.floor(Math.random() * result.length)];
            var randomGuide = '<a href="' + entry.url + '">' + entry.name + '</a>';
            $('#foo').html(randomGuide);
  });  });  });
 {% endhighlight %}

* Line 2: foo1 and foo2 are placeholders. Fill these in with the information provided by Springshare.
* Line 2: ```status=1``` pulls JSON data for only your published guides.
* Line 3: This is the randomizing part.
* Line 4: Builds the actual link pulling in the url and name from the JSON data returned from the API call.
* Line 5: Attaches to the ```#foo``` and places the link we've built.
* Line 7: This is the beginning of what fuels the button which will regenerate the random link.
* Lines 8-11: Essentially the same as the above.         