---
layout: implementations
title: "Dynamic New Books Display"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
tags:
- javascript
- jquery
- primo
---

## Description

Do you have a subject heading for your new books? Would you like to display them on your website or libguide, with book cover images? Would you like to never have to update the page with newer new books? Then this is for you!

Every time a user loads the page our code will query Primo for our books labeled with our "new books" subject heading and will then randomly select a few (you determine how many) from that group. It will then query Google Books for a cover image for the book (and will not display the book without a cover image). Then it places them on the page for the user. We use an animated gif preloader to show the user that something is happening while these operations complete.

Also included here are styles that will allow a hover effect as shown in the screenshot, that displays the books title. Clicking any book will take you to its record in Primo. 


### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

### More details

One prerequisite for doing this is having an API key setup for your institution's Primo. You can find instructions on this page: [Primo REST Apis](https://developers.exlibrisgroup.com/primo/apis/). That might be the hardest part of implementing this. If not, the hardest part will be determining the exact query to use in the call to the API, as far as how to refer to your new books subject heading. 

This code is all vanilla javascript except for the API calls which use jQuery for stability and cross-browser support. Libguides and most school CMSes are already loading jQuery so this shouldn't be a problem. If you're using this code somehwere without jQuery you will need to either add it to the page or rewrite the code without it. 


## The Code

#### HTML
{:.code-notes}

* You'll need to change the source of the image. I found our preloader on [Preloaders.net](https://icons8.com/preloaders/).

{% highlight html linenos %}

   <div id="new-books-1">
      <img
        src="https://www.example.com/preloader1.gif"
        alt="preloader"
        id="library-preloader"
      />
      <ul id="new-books"></ul>
    </div>

{% endhighlight %}

#### CSS
{:.code-notes}

* The first four rules here are adding a frame around the images and reigning in their size. The rest is for the nifty hover effects.

{% highlight css linenos %}
#new-books-1 .book-cover {
    box-shadow: 0px 0px 2px darkgray;
}
#new-books-1 .new-books-li {
    box-shadow: 1px 1px 3px lightgray;
    padding: 1rem;
}
#new-books-1 ul li {
    display: inline-block;
    margin-right: 1rem;
    margin-bottom: 1rem;
}
#new-books-1 .book-cover {
    max-width: 145px;
    max-height: 215px;
}
 
.content {
    position: relative;
    max-width: 400px;
    margin: auto;
    overflow: hidden;
}
.content .content-overlay {
    background: rgba(0, 0, 0, 0.8);
    position: absolute;
    height: 99%;
    width: 100%;
    left: 0;
    top: 0;
    bottom: 0;
    right: 0;
    opacity: 0;
    -webkit-transition: all 0.4s ease-in-out 0s;
    -moz-transition: all 0.4s ease-in-out 0s;
    transition: all 0.4s ease-in-out 0s;
}
.content:hover .content-overlay {
    opacity: 1;
}
.content-image {
    width: 100%;
}
.content-details {
  
    position: absolute;
    text-align: center;
    padding-left: 5px;
    padding-right: 5px;
    width: 100%;
    top: 50%;
    left: 50%;
    opacity: 0;
    -webkit-transform: translate(-50%, -50%);
    -moz-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
    -webkit-transition: all 0.3s ease-in-out 0s;
    -moz-transition: all 0.3s ease-in-out 0s;
    transition: all 0.3s ease-in-out 0s;
}
.content:hover .content-details {
    top: 50%;
    left: 50%;
    opacity: 1;
}
.content-details {
    font-family: "Roboto Condensed", sans-serif;
}
.content-details .content-title {
    color: #fff;
    font-weight: 500;
    text-transform: uppercase;
    font-size: 14px;
    padding: 5px;
}
.content-text .fa-external-link-alt {
    font-size: 2.1rem;
}
.content-details p {
    color: #fff;
    font-size: 1.2rem;
}
.fadeIn-bottom {
    top: 80%;
}
p.content-text {
    margin-bottom: 1px;
}

{% endhighlight %}

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}

* Line 5: `lsr03%2Cexact%2Cnewbooks`..."lsr03" is doing a subject search (I thought it was going to be something different, but this works) and "newbooks" is the subject heading for our new books.
* Line 5: `vid=01TRAILS_ROCKY` Change this to what is correct for your system. If you go to your Primo advanced search screen, you'll see this in the URL.
* Line 5: `YOUR_API_KEY_HERE` Instructions for getting setup with an API key can be found at [Primo REST Apis](https://developers.exlibrisgroup.com/primo/apis/)
* Line 5: `limit=100` is how many books you want to grab from Primo. It's a highish number so that we have plenty to work with as far as randomly selecting things.
* Line 5: `scope=P-01TRAILS_ROCKY` can be found in your Primo backend. 
* Line 29: The `10` here should be at least a few higher than the total number of books you hope to display. This gives us wiggle room for books that don't have cover images.
* Line 62: The `7` here is how many books you ultimately want displayed.

##### The code itself:

{% highlight javascript linenos %}
let newBooksDynamism = (function() {
  let totalDisplayed = 0;
  //Getting a list of books with the subject newbooks limit=100 is one hundred results to work with.
  $.getJSON(
    "https://api-na.hosted.exlibrisgroup.com/primo/v1/search?q=lsr03%2Cexact%2Cnewbooks&vid=01TRAILS_ROCKY&tab=default_tab&limit=100&scope=P-01TRAILS_ROCKY&apikey=YOUR_API_KEY_HERE",
    function(result) {
      var jsonContents = result.docs;
      var jsonResponseLength = jsonContents.length;

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
      var ourRandoms = getRandomNumbers(10, jsonResponseLength);

      bookCoverGrab(result, ourRandoms); //call a function with the full results from the API call and our random numbers.
    }
  );
  //function to get book cover image url strings
  function bookCoverGrab(input, randos) {
    for (i = 0; i < randos.length; i++) {
      let theIsbn = input.docs[randos[i]].pnx.search.isbn[0];
      let theTitle = input.docs[randos[i]].pnx.display.title;
      let theCatalogLink = `<a href="https://rocky-primo.hosted.exlibrisgroup.com/permalink/f/1j18u99/${input.docs[randos[i]].pnx.control.sourceid}${input.docs[randos[i]].pnx.control.sourcerecordid}"
                target="_blank">`;

      $.getJSON(
        `https://books.google.com/books?bibkeys=ISBN:${theIsbn}&jscmd=viewapi&callback=?`,
        function(data) {
          if (data[`ISBN:${theIsbn}`].thumbnail_url != undefined) {
            addToDom(
              data[`ISBN:${theIsbn}`].thumbnail_url,
              theTitle,
              theCatalogLink
            );
            $("#library-preloader").hide();
          } else {
            console.log("No ISBN", i);
          }
        }
      );
    }
  }

  function addToDom(theIMG, theTitle, catalogLink) {
    //change the 7 to however many you want to display
    if (totalDisplayed < 7) {
      class RmcNewBooks {
        constructor(theBookStuff) {
          this.theBookStuff = theBookStuff;
        }
        getToAppending() {
          var domsn = document.getElementById("new-books");
          domsn.insertAdjacentHTML("beforeend", this.theBookStuff);
          totalDisplayed++;
        }
      }
      var theBookStuff = `
      <li class="new-books-li">
        <div class="content">
         
            ${catalogLink}
                <div class="content-overlay"></div>
                <img class="content-image book-cover" src="${theIMG}">
                <div class="content-details fadeIn-bottom">
                    <div class="content-title">${theTitle}
                     
            
                </div>
            </a>
        </div>
    </li>`;

      var ttttt = new RmcNewBooks(theBookStuff);
      ttttt.getToAppending();
    }
  }

  // }
})();
{% endhighlight %}
