---
layout: implementations
title: "Antislavery Movements"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com" 
tags: text-collections
---

## Description

The [Internet Archive](https://www.archive.org/) houses a collection of more than 350 texts with the subject of ["Antislavery movements -- United States"](https://archive.org/search.php?query=subject%3A%22Antislavery+movements+--+United+States%22). The code below grabs a number of random items and displays them in a nice way. It's very easy to specify how many items you want displayed.

Something like this can help promote seredipitous discovery in your research
guides. It also has the benefit of being something that is fresh and constantly
changing without you needing to constantly update it.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

### More details

The styles included here use flexbox to display each book image to the left of the text. It also vertically centers the text. On small screens it will adjust to have the text below the image.

Currently everything is displayed in one vertical column, book after book. It would be possible to have books in multiple vertical columns but doing that would depend on the code situation of where you were using it. If you want multiple columns I'm willing to help, just contact the project using the form in the footer of this page.

## The Code

#### HTML

{:.code-notes}

* Just the blank `<ul>` our JS will work with.
  {% highlight html linenos %}
  
    <ul id="antislavery-texts"></ul>{% endhighlight %}

#### CSS

{:.code-notes}

* This CSS just uses flexbox to position the image to the left and the text to
  the right.
* The media query makes it so that on small screens the text switches to below
  the image.

{% highlight css linenos %} .flexy-container {
display: flex;
align-items: center;
}
.flexy-container .left {
padding: 1rem;
}
@media (max-width: 500px) {
.flexy-container {
flex-wrap: wrap;
}
.flexy-container img {
display: block;
margin: 0 auto;
}
}
.flexy-container a {
text-decoration: dotted underline gray;
color: #333;
}
.AS-title {
font-weight: bold;
font-size: 1.1em;
} {% endhighlight %}

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}

* Line 22: `getRandomNumbers(4, jsonResponseLength);` To change the number of items displayed change the `4` to any number that doesn't exceed the amount of results from the API.


##### The code itself:

{% highlight javascript linenos %} let APIS4LIBS_antislaverymovements = function(){$(document).ready(function() {  $.getJSON( "https://archive.org/advancedsearch.php?q=subject%3A(%22Antislavery+movements+--+United+States%22)&fl[]=creator&fl[]=identifier&fl[]=title&fl[]=year&sort[]=&sort[]=&sort[]=&rows=400&page=1&output=json&callback=?", function(data) { console.log(data);   let jsonContents = data.response.docs; let jsonResponseLength = data.response.docs.length;

    
    var getRandomNumbers = function(howMany, upperLimit) {
      var limit = howMany,
        amount = 1,
        lower_bound = 1,
        upper_bound = upperLimit,
        unique_random_numbers = [];
      if (amount > limit) limit = amount;   
      while (unique_random_numbers.length < limit) {
        var random_number = Math.floor(
          Math.random() * (upper_bound - lower_bound) + lower_bound
        );
        if (unique_random_numbers.indexOf(random_number) == -1) {
          unique_random_numbers.push(random_number);
        }
      }
      return unique_random_numbers;
    };
    //This is where we actually specify how many random numbers (and therefore how many books) we want generated.
    var ourRandoms = getRandomNumbers(4, jsonResponseLength);

    class AntiSlaveryBook {
      constructor(theBookStuff) {
        this.theBookStuff = theBookStuff;
      }
      getToAppending() {
        var domsn = document.getElementById("antislavery-texts");
        domsn.insertAdjacentHTML("beforeend", this.theBookStuff);
      }
    }
    for (i = 0; i < ourRandoms.length; i++) {
      let theBookStuff = `<li class=flexy-container><div class="left"><div class="AS-pic"><a href="https://archive.org/details/${
        jsonContents[ourRandoms[i]].identifier
      }"><img src="https://archive.org/services/img/${
        jsonContents[ourRandoms[i]].identifier
      }"></a></div></div><div class="right">
    <div class="AS-title"><a href="https://archive.org/details/${
      jsonContents[ourRandoms[i]].identifier
    }">${jsonContents[ourRandoms[i]].title}</a></div>
    <div class="AS-creator">Creator(s): ${
      jsonContents[ourRandoms[i]].creator
    }</div>
    <div class="AS-published">Published: ${
      jsonContents[ourRandoms[i]].year
    }</div>

    </div>
    </li>`;

      let ttttt = new AntiSlaveryBook(theBookStuff);
      ttttt.getToAppending();
    }
     
  }
);

});};APIS4LIBS_antislaverymovements();
{% endhighlight %}
