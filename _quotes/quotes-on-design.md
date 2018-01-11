---
layout: implementations
title: "Random Quote - Quotes on Design"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
tags: quotesdesign
---

## Description

[Quotes on Design](https://quotesondesign.com/) is a great site that displays one quote at a time by visual and web designers. The collection is curated by Chris Coyier of [CSS-Tricks](https://css-tricks.com/) and [CodePen](https://codepen.io/#) fame.

The code below will query their API and display a new quote each time the page its on is reloaded. Have fun!

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}
![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot-1.jpg){:.screenshot}

### More details

The code below utilizes JSONP and should be compatible with even old Internet Explorers. You can style things however you like. The screenshot is just a simple example.

## The Code

#### HTML

{% highlight html linenos %}

  <div id="design-quote"></div>
{% endhighlight %}

#### CSS

{:.code-notes}

{% highlight css linenos %}

.design-quote-quote {font-family: 'Martel';line-height: 1.4rem;}
.design-quote-quote a {text-decoration: none;color: #222;}
.design-quote-person {color: #333;}
#design-quote {max-width: 480px;}

{% endhighlight %}

#### JavaScript

##### Notes for implementation:

{:.code-notes}

* You don't have to change anything in this code for it to work. There is no special authentication necessary.

##### The code itself:

{% highlight javascript linenos %}
var apis4librarians_quotesOnDesign = (function() {
  function reqListener() {
    var data = JSON.parse(this.responseText);
    var d1 = document.getElementById("design-quote");
    d1.insertAdjacentHTML(
      "beforeend",
      `<div class="design-quote-quote"><a href="${data[0].link}">${
        data[0].content
      }</a></div><div class="design-quote-person">--${data[0].title}</div>`
    );
  }
  var oReq = new XMLHttpRequest();
  oReq.addEventListener("load", reqListener);
  oReq.open(
    "GET",
    "http://quotesondesign.com/wp-json/posts?filter[orderby]=rand&filter[posts_per_page]=1"
  );
  oReq.send();
})();
{% endhighlight %}
