---
layout: implementations
title: "Suggest Random Database"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
tags:
- javascript
- jquery
- requires-springshare-auth-server
- client-side
---

## Description

This will display a link to a random database from your AZ list in LibGuides. You can narrow your results by subject if you like. Every time the page is loaded this code will show a different database.

You might wish to place this on a research guide or even in your discovery tool. FYI, you don't need to include the text "Have you considered?" You can easily just use the database link itself.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

### More details

{:.auth-server-promo}
Works with: [APIs for Librarians: Springshare Auth Server]({{site.baseurl}}/servers/springshare-auth-server)

The API call, which is written in jQuery, sends a request to a proxy server that handles authorization for the API and then grabs the data and then returns it to our code. You're welcome to use the Springshare Auth Server mentioned just above or roll your own.

The code takes a look at the returned list of databases and randomly selects one to display. You can easily limit the possible databases to specific subjects or it can be from your entire AZ list.

## The Code

#### HTML

{:.code-notes}

* A container to help us style it, a header, and the database itself. The only essential thing is `#random-database`

{% highlight html linenos %}

<div id="random-database-container">
    <h4 id="random-database-header">Have you considered?</h4>
    <div id="random-database"></div>
</div>
{% endhighlight %}

#### CSS

{:.code-notes}

* Reminder: All styles optional. One of the great things about using APIs is the control and flexibility you have over styling.

{% highlight css linenos %}
#random-database-container {font-family: 'Martel', serif;font-size: 1.2em;text-align: center;}
#random-database-header {margin-bottom: 0px;}
#random-database {margin: 10px 0;}
#random-database a {text-decoration: none;}
{% endhighlight %}



#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}

* Line 4: `YOUR_SERVER_HERE.herokuapp.com` will be replaced with your Heroku server's URL or whatever proxy server you're using to handle authentication. It's free and easy to use the one I built for you: [APIs for Librarians: Springshare Auth Server]({{site.baseurl}}/servers/springshare-auth-server)
* Line 4: `/az?subject_ids=41905` can be customized based on the requirements of the LibGuides API. If you want to work with all of your AZ List you can make this: `/az`

##### The code itself:

{% highlight javascript linenos %}var apis4librarians_randomDatabase = (function() {
var getRandomDatabase = function() {
$.getJSON(
"https://YOUR_SERVER_HERE.herokuapp.com/springshare/libguides/passthrough?what=/az?subject_ids=41905",
function(result) {
console.log(result);
var entry = result[Math.floor(Math.random() * result.length)];
var randomDatabase = '<a href="' + entry.url + '">' + entry.name + "</a>";
$("#random-database").html(randomDatabase);
}
);
};
getRandomDatabase();
})();

{% endhighlight %}
