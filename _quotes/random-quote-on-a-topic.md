---
layout: implementations
title: "Random Quote on a Topic"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
---
## Description
        
This checks the status of one of your LibChat widgets and based on a true/false response displays custom text. So, if the widget is online (meaning that a particular chat queue is active) you can have it say whatever you want. In my example I have ```Online!``` and ```Offline```, respectively.

### Screenshot

The styling here is more just a proof-of-concept. But you have the full range of CSS and JS at your disposal to make it look as you want.

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}
![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot2.jpg){:.screenshot}

       
### More details
The API call, which is written in jQuery, will send a widget ID to your LibChat system and get a response as to whether or not that particular widget is online. You can use this to monitor general chat queues or the chat queues automatically created for each LibChat librarian.
        
The code is just a simple if/else that checks to see if LibChat has responded ```true```. If so, you can have it display one thing and if not then another. The success response can be linked to a pop-out full chat interface and the fail response can be linked to your LibAnswers system. Or they can not be linked. Or linked to anything you'd like.

SpringShare does have a button widget that does essentially the same thing as this and, in fact, they do allow heavy customization of it. Though doing it this way gives you complete control over it.
    
## The Code

#### HTML/CSS

{% highlight html linenos %}

{% endhighlight %}

{% highlight css linenos %}

{% endhighlight %}

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}
* Line ?: 
* Line ?: 
* Line ?: 

##### The code itself:
{% highlight javascript linenos %}
$(document).ready(function() {
  //converts XML to JSON
  function xmlToJson(xml) {
    var obj = {};
    if (xml.nodeType == 1) {
      if (xml.attributes.length > 0) {
        obj["@attributes"] = {};
        for (var j = 0; j < xml.attributes.length; j++) {
          var attribute = xml.attributes.item(j);
          obj["@attributes"][attribute.nodeName] = attribute.nodeValue;
        }
      }
    } else if (xml.nodeType == 3) {
      obj = xml.nodeValue;
    }
    if (xml.hasChildNodes()) {
      for (var i = 0; i < xml.childNodes.length; i++) {
        var item = xml.childNodes.item(i);
        var nodeName = item.nodeName;
        if (typeof obj[nodeName] == "undefined") {
          obj[nodeName] = xmlToJson(item);
        } else {
          if (typeof obj[nodeName].push == "undefined") {
            var old = obj[nodeName];
            obj[nodeName] = [];
            obj[nodeName].push(old);
          }
          obj[nodeName].push(xmlToJson(item));
        }
      }
    }
    return obj;
  }
  //Gets a random integer.
  function getRandomIntInclusive(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }
  var getRandomQuote = function() {
    $.get('http://www.stands4.com/services/v2/quotes.php?uid=5722&tokenid=ReVtjomO3okWzsHs&searchtype=SEARCH&query=writing', function (data) {
      //xmltojson
      var funTimes = xmlToJson(data);
      //Creating a variable going deeper into the JSON object to make later code neater
      var ohGee = funTimes.results.result;
      //Cleans up the output of the author and quote because without this there's some weirdness
      var processAuthor = function(data) {
        var s = JSON.stringify(data);
        var trimEnd = s.substr(0, s.length - 2);
        var trimStart = trimEnd.substr(10);
        return trimStart;
      };
      var processQuote = function(data) {
        var m = JSON.stringify(data);
        var ttrimEnd = m.substr(0, m.length - 1);
        var ttrimStart = ttrimEnd.substr(9);
        return ttrimStart;
      };
      //Selects and displays the quote/author. If the quote is longer than 145 characters it will select another quote to display.
      function quoteSelection(quoteArray) {
        var randomLength = getRandomIntInclusive(0, quoteArray.length);
        var theQuoteItself = processQuote(quoteArray[randomLength].quote);
        var theAuthor = processAuthor(quoteArray[randomLength].author);
        if (theQuoteItself.length < 145) {
          $("#random-quote").append(
            '<div id="quote-wrapper"><div id="quote-text">' +
              theQuoteItself +
              "</div>" +
              '<div id="quote-author">' +
              " -" +
              theAuthor +
              "</div></div>"
          );
        } else {
          quoteSelection(ohGee);
        }
      }
      quoteSelection(ohGee);
    });
  };
  getRandomQuote();
});
{% endhighlight %}

