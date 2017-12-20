---
layout: implementations
title: "Word of the day"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
---
## Description
        
Love language? Think words are cool? Me too! 

[Wordnik](http://wordnik.com/) publishes a word of the day and this code grabs that and displays it for your users. Every aspect of the HTML has an id or class to make it easy for you to style it any way that you would like.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}
       
### More details

In order to use the Wordnik API you need to sign up for a free account and then request an API key at their [developer site](http://developer.wordnik.com/). You will be given an API key (aka a long string of gibberish) that you will fit into the code provided below. It's a simple process.

Some words have multiple definitions. This implementation only shows the primary definition.
    
## The Code

#### HTML

{% highlight html linenos %}
<h3 id="wordnik-wordofday-heading">Word of the day!</h3>
<div id="wordnik-wordofday"></div>
{% endhighlight %}

#### CSS

{:.code-notes}
* None of the CSS here is essential to the functionality. It's just some simple styling to make the screenshot look better :).

{% highlight css linenos %}
body{font-family: 'Martel'}
.wordnik-word{font-weight: 800;font-size: 1.1em; font-style: italic;}
.wordnik-word-partofspeech{font-size: .7rem;font-weight: 400;}
#wordnik-wordofday {padding-left:15px;margin-top: 0px;}
#wordnik-wordofday a {color: inherit;text-decoration-line: none;}
#wordnik-wordofday-heading{margin-bottom: 0px;}
{% endhighlight %}

#### JavaScript

##### Notes for implementation:

{:.code-notes}
* Line 24: Replace `YOUR_API_KEY_HERE` with the API key given to you by Wordnik. Acquire it at their [developer site](http://developer.wordnik.com/).

##### The code itself:
{% highlight javascript linenos %}let apis4librarians_wordnikofdaySimple = function(){function reqListener() {
  let data = JSON.parse(this.responseText);
  console.log(data);

  var d1 = document.getElementById("wordnik-wordofday");
  d1.insertAdjacentHTML(
    "beforeend",
    `
     <span class="wordnik-word"><a href="https://www.wordnik.com/words/${data.word}">${
        data.word
      }  </a></span><span class="wordnik-word-partofspeech"> ${
      data.definitions[0].partOfSpeech
    }</span> 
      <br><span class="wordnik-word-definition">${
        data.definitions[0].text
      }</span>
       `
  );
}
var oReq = new XMLHttpRequest();
oReq.addEventListener("load", reqListener);
oReq.open(
  "GET",
  "http://api.wordnik.com:80/v4/words.json/wordOfTheDay?api_key=YOUR_API_KEY_HERE"
);
oReq.send();
}();
{% endhighlight %}

