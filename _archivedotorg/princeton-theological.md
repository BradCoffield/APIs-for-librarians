---
layout: implementations
title: "Princeton Theological Seminary Books"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com" 
tags: text-collections
---

## Description

The [Internet Archive](https://www.archive.org/) houses a collection of more than [73k texts](https://archive.org/details/Princeton) from the [Princeton Theological Seminary](https://library.ptsem.edu/). The code below grabs a random number of items and displays them in a nice way. It's very easy to specify how many items you want displayed.

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

* Basically just our spinny preloader (which isn't essential) and the blank `<ul>` our JS will work with. 
  {% highlight html linenos %}
    <img src="/assets/img/Eclipse.gif" alt="" id="preloader">
    <ul id="IA-princeton-theological"></ul>{% endhighlight %}

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
.flexy-container a {
  text-decoration: dotted underline gray;
  color: #333;
}
.flexy-container .left {padding: 1rem;}
.pb-title {
  font-weight: bold;
  font-size: 1.1em;
}
@media (max-width: 500px) {
  .flexy-container {flex-wrap: wrap;}
  .pb-pic img {
    display: block;
    margin: 0 auto;
  }
}


{% endhighlight %}

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}

* Line 32: `getRandomNumbers(4, jsonResponseLength);` To change the number of items displayed change the `4` to any number that doesn't exceed the amount of results from the API.
* Line 6: This is our request to the API. To change the amount of results, modify `rows=1000` to a different number. Note, the code as it is gets 1000 and this is simply how many of the collection's items are available to us to pick from (for randomly displaying individual items). 

##### The code itself:

{% highlight javascript linenos %} $(document).ready(function() {
  //Show our spinny preloader
  document.getElementById("preloader").style.display = "";
  //Gets the data
  $.getJSON(
    "https://archive.org/advancedsearch.php?q=collection%3Aprinceton+mediatype%3Atexts&fl[]=creator&fl[]=description&fl[]=identifier&fl[]=publisher&fl[]=subject&fl[]=title&fl[]=volume&fl[]=year&sort[]=&sort[]=&sort[]=&rows=1000&page=1&output=json&callback=?",
    function(data) {
      //Removes our preloader
      document.getElementById("preloader").style.display = "none";
      let jsonContents = data.response.docs;
      let jsonResponseLength = data.response.docs.length;

      //This is the function to generate as many random numbers we want - with the amount of API results as the upper range.
      var getRandomNumbers = function(howMany, upperLimit) {
        var limit = howMany,
          amount = 1,
          lower_bound = 1,
          upper_bound = upperLimit,
          unique_random_numbers = [];
        if (amount > limit) limit = amount; //Infinite loop if you want more unique natural numbers than exist in a given range
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

      class PrincetonTheologicalBook {
        constructor(theBookStuff) {
          this.theBookStuff = theBookStuff;
        }
        getToAppending() {
          var domsn = document.getElementById("IA-princeton-theological");
          domsn.insertAdjacentHTML("beforeend", this.theBookStuff);
        }
      }
      for (i = 0; i < ourRandoms.length; i++) {
        let theBookStuff = `<li class=flexy-container><div class="left"><div class="pb-pic"><a href="https://archive.org/details/${
          jsonContents[ourRandoms[i]].identifier
        }"><img src="https://archive.org/services/img/${
          jsonContents[ourRandoms[i]].identifier
        }"></a></div></div><div class="right">
      <div class="pb-title"><a href="https://archive.org/details/${
        jsonContents[ourRandoms[i]].identifier
      }">${jsonContents[ourRandoms[i]].title}</a></div>
      <div class="pb-creator">Creator(s): ${
        jsonContents[ourRandoms[i]].creator
      }</div>
      <div class="pb-published">Published: ${
        jsonContents[ourRandoms[i]].year
      }</div>
      <div class="pb-subjects">Subject(s):  <span class="pb-sub-span">${
        jsonContents[ourRandoms[i]].subject
      }</span></div>
      </div>
      </li>`;

        let ttttt = new PrincetonTheologicalBook(theBookStuff);
        ttttt.getToAppending();
      }
      //Check if any subjects are undefined and if so replace with custom text
      var princetonSubjects = document.getElementsByClassName("pb-sub-span");
      const descMessage = "Unavailable.";
      for (let element of princetonSubjects) {
        if (element.innerHTML == "undefined" || undefined) {
          element.innerHTML = descMessage;
        }
      }
    }
  );
});

{% endhighlight %}
