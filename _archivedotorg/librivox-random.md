---
layout: implementations
title: "Librivox Audiobooks"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com" 
tags: internet-archive audio-and-video-collections
---

## Description

[Librivox](https://archive.org/details/librivoxaudio) is an amazing project where volunteers create audiobooks of public domain texts. It currently features more than eleven thousand titles! You can use these items to bolster research guides on specific authors (like Shakespeare, who is featured in the screenshot below), or topical guides on literature, philosophy, religious studies, and more.

This code asks the [archive.org](https://www.archive.org/) API for a list of [Librivox Audiobooks](https://archive.org/details/librivoxaudio). You get to specify the topic (as a keyword search). Then we display a number of results (you decide how many). Those items are randomly selected from the list of audiobooks from the Librivox collection. Each time the page is loaded new items are displayed.

It's also possible to not specify a search and simply choose from amongst all Librivox books. In addition to that you can limit your results by download count - you may wish to set the bar high to only return things many have downloaded.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

### More details

The styles included here use flexbox to display each book image to the left of the text. It also vertically centers the text. On small screens it will adjust to have the text below the image.

Currently everything is displayed in one vertical column. It would be possible to have items in multiple vertical columns but doing that would depend on the code situation of where you were using it. If you want multiple columns, I'm willing to help so just contact the project using the form in the footer of this page.

Descriptions are started but then hidden behind a 'Read More/Less' link which, when
clicked, will expand and show the rest of the description. Some descriptions are very long and our code helps ensure a consistent visual style while still giving the use access the information they may desire.

## The Code

#### HTML

{:.code-notes}

* Just our spinny preloader (which isn't necessary) and the `<ul>` that will be populated by our javascript

{% highlight html linenos %}<img src="/assets/img/Eclipse.gif" alt="" id="preloader" style="display:none;">

<ul id="librivox"></ul>
{% endhighlight %}

#### CSS

{:.code-notes}

* This CSS just uses flexbox to position the image to the left and the text to
  the right.
* The media query makes it so that on small screens the text switches to below
  the image.
* `.ellip` is necessary to have the description hiding/showing.
* You probably have your fonts set but the screenshot uses "Martel" which is available from Google Fonts.

{% highlight css linenos %}body {
font-family: "Martel";
color: #333;
font-size: 16px;
}
.flexy-container {
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
.librivox-title {
font-weight: bold;
font-size: 1.1em;
}
.ellip {
white-space: nowrap;
overflow: hidden;
text-overflow: ellipsis;
width: 550px;
}
{% endhighlight %}

#### JavaScript

##### Notes for implementation:

{:.code-notes}

* Line 8: This is our request to the archive. We're concerned with `q=shakespeare+downloads%3A[1+TO+1000000000]+collection%3Alibrivoxaudio` We can modify a couple things in this line.
  * To change the topic: Replace `shakespeare` with something else. You can't use actual spaces here. If you want multiple words put a + between them e.g. `william+shakespeare`.
  * To remove the topic: You can remove Shakespeare altogether so it would be `q=downloads`
  * To only get popular things:
* Line 38: `getRandomNumbers(4, jsonResponseLength);` To change the number of items displayed change the `4` to any number that doesn't exceed the amount of results from the API.
* Code updated 12/14/17 to use pure JavaScript and properly namespace itself.

##### The code itself:

{% highlight javascript linenos %} var apis4librarians_librivoxRandom = (function() {
//Show our spinny preloader
document.getElementById("preloader").style.display = "";
//Gets the data
var scriptEl = document.createElement("script");
scriptEl.setAttribute(
"src",
"https://archive.org/advancedsearch.php?q=shakespeare+downloads%3A[1+TO+1000000000]+collection%3Alibrivoxaudio&fl[]=creator&fl[]=description&fl[]=downloads&fl[]=identifier&fl[]=title&sort[]=&sort[]=&sort[]=&rows=1000&page=1&output=json&callback=myJsonpCallback"
);
document.body.appendChild(scriptEl);

window.myJsonpCallback = function(data) {
console.log(data);
//Removes our preloader
document.getElementById("preloader").style.display = "none";
varjsonContents = data.response.docs;
varjsonResponseLength = data.response.docs.length;

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

    class LibrivoxBook {
      constructor(theBookStuff) {
        this.theBookStuff = theBookStuff;
      }
      getToAppending() {
        var domsn = document.getElementById("librivox");
        domsn.insertAdjacentHTML("beforeend", this.theBookStuff);
      }
    }
    for (i = 0; i < ourRandoms.length; i++) {
      vartheBookStuff = `<li class=flexy-container><div class="left"><div class="librivox-pic"><a href="https://archive.org/details/${
        jsonContents[ourRandoms[i]].identifier
      }"><img src="https://archive.org/services/img/${
        jsonContents[ourRandoms[i]].identifier
      }"></a></div></div><div class="right">

 <div class="librivox-title"><a href="https://archive.org/details/${
   jsonContents[ourRandoms[i]].identifier
 }">${jsonContents[ourRandoms[i]].title}</a></div>
 <div class="librivox-creator">Creator(s): ${
   jsonContents[ourRandoms[i]].creator
 }</div>
 <div class="librivox-description">Description:  <div class="item-description-content ellip" style="margin-left:1rem;">${
   jsonContents[ourRandoms[i]].description
 }</div><a class="description-toggler" style="cursor:pointer;font-size:11px;margin-left:1rem;">Read more/less</a></div>

 </div>
 </li>`;
      //Builds a new object for each item and then places it on our webpage.
      varttttt = new LibrivoxBook(theBookStuff);
      ttttt.getToAppending();
    }
    //This is where our description hiding/showing is happening.
    document.body.addEventListener("click", e => {
      if (!e.target.classList.contains("description-toggler")) return;
      console.log(e.target.nextSibling);
      e.target.previousSibling.classList.toggle("ellip");
    });
  };
})();

{% endhighlight %}
