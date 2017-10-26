---
layout: implementations
title: "Random Quote on a Topic"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
---
## Description
        
This queries the [Quotes.net](http://www.quotes.net/) API and returns a list of quotes that match your keyword search. You can specify the terms of the search in the javascript code. Each time the page is loaded a random quote is selected from the returned batch and will be displayed. The code is set to reject any quotes that are more than 145 characters but you can easily adjust this number up or down. 

### Screenshot

You can style it however you want. This is just an example.

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}
       
### More details

In order to use the Quotes.net API you need to sign up for a free account. Free accounts can query the API up to 100 times per day. Therefore, it's likely not suitable for a library homepage or widespread usage across libguides. However, usage on a few guides or pages with less traffic would be just fine.

Details on acceptable use of the API and to sign up can be found at [http://www.quotes.net/quotes_api.php](http://www.quotes.net/quotes_api.php).

Another option would be to create your own set of quotes and build them into a JSON file you host on your own servers. This would have the benefit of unlimited usage and personalized curation. You would only need to change the path in the code below, which is noted.

    
## The Code

#### HTML

{% highlight html linenos %}
<div id="random-quote"></div>
{% endhighlight %}

#### CSS

{:.code-notes}
* Line 19: `#quote-wrapper` isn't necessary for production but gives the quote a specific space to inhabit.

{% highlight css linenos %}
#quote-text {
    font-size: 1.5em;
    line-height: 1.2em;
    letter-spacing: .01em;
    font-family: 'Martel', serif;
    text-shadow: 1px 1px lightgray;
    font-weight: bold;
    text-align: right;
}

#quote-author {
    font-size: 1em;
    font-family: 'Martel', serif;
    font-weight: 400;
    text-align: right;
    font-style: italic;
}

#quote-wrapper {
    max-width: 20em;
}
{% endhighlight %}

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}
* Line 41: `uid` and `tokenid` will be given to you by Quotes.net once you sign up. 
* Line 41: `query=writing` is where you put your keyword to search. Replace `writing` with whatever suits you.
* Line 41: If you wanted to use a handcrafted JSON file you would change the long string that starts with `http://` with the path to your local file. Something like `'/quotes.json'`
* Line 64: Change `theQuoteItself.length < 145` to some other number if you want to allow longer quotes or restrict it to shorter quotes. 

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
    $.get('http://www.stands4.com/services/v2/quotes.php?uid=0000&tokenid=0000&searchtype=SEARCH&query=writing', function (data) {
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

