---
layout: implementations
title: "Prelinger Archives"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com" 
tags: prelinger audio-and-video-collections
---

## Description

[The Prelinger Archives](https://archive.org/details/prelinger&tab=collection) is an awesome resource featuring ["a collection of over 60,000 'ephemeral' (advertising, educational, industrial, and amateur) films."](https://archive.org/details/prelinger&tab=about)

The code below filters their content for things that are highly rated by users and selects a number of items for display (you dictate how many). On each page load new items are selected and displayed.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

### More details

Overly long item descriptions are being hidden behind a link ("read more"). When
clicked that will expand and show the rest of the description. I decided to do
this because some of the descriptions are quite long. 

The code is vanilla javscript except for the $.getJSON piece from jQuery.

## The Code

#### HTML

{:.code-notes}

* I downloaded an animated gif spinner that will display while the API
  is working and before the results have loaded. You don't have to do this but
  it does give a visual cue to the user that something is happening behind the
  scenes. 
  
  {% highlight html linenos %}
  <img src="/assets/img/Eclipse.gif" alt="" id="preloader" style="display:none;">
  <ul id="prelinger"></ul>

{% endhighlight %}

#### CSS

{:.code-notes}

* This CSS just uses flexbox to position the image to the left and the text to
  the right.
* The media query makes it so that on small screens the text switches to below
  the image.

{% highlight css linenos %} .flexy-container {
  display: flex;
  align-items: center;
  margin-bottom: .75em;
  font-family: "Martel";
  color: #333;
  font-size: 16px;
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
.prelinger-title {
  font-weight: bold;
  font-size: 1.1em;
}
.prelinger-pic img {
  max-width: 180px;
}{% endhighlight %}

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}

* Lines 20-180: This part of code is what does the description hiding and is
  from [http://viralpatel.net](http://viralpatel.net).
* Lines 196-205: Could be removed if you only want to display one result. If so
  then just add `let num1= getRandomIntInclusive(0, data.response.docs.length);`
  in their stead.
* Lines 214-227: Also delete these if you only want one result.

##### The code itself:

{% highlight javascript linenos %} 
/*
 * jQuery Shorten plugin 1.1.0
 *
 * Copyright (c) 2014 Viral Patel
 * http://viralpatel.net
 *
 * Licensed under the MIT license:
 *   http://www.opensource.org/licenses/mit-license.php
 */

/*
** updated by Jeff Richardson
** Updated to use strict,
** IE 7 has a "bug" It is returning undefined when trying to reference string characters in this format
** content[i]. IE 7 allows content.charAt(i) This works fine in all modern browsers.
** I've also added brackets where they weren't added just for readability (mostly for me).
*/
(function($) {
  $.fn.shorten = function(settings) {
    "use strict";

    var config = {
      showChars: 100,
      minHideChars: 10,
      ellipsesText: "...",
      moreText: "more",
      lessText: "less",
      onLess: function() {},
      onMore: function() {},
      errMsg: null,
      force: false
    };

    if (settings) {
      $.extend(config, settings);
    }

    if ($(this).data("jquery.shorten") && !config.force) {
      return false;
    }
    $(this).data("jquery.shorten", true);

    $(document).off("click", ".morelink");

    $(document).on(
      {
        click: function() {
          var $this = $(this);
          if ($this.hasClass("less")) {
            $this.removeClass("less");
            $this.html(config.moreText);
            $this
              .parent()
              .prev()
              .animate({ height: "0" + "%" }, function() {
                $this
                  .parent()
                  .prev()
                  .prev()
                  .show();
              })
              .hide("fast", function() {
                config.onLess();
              });
          } else {
            $this.addClass("less");
            $this.html(config.lessText);
            $this
              .parent()
              .prev()
              .animate({ height: "100" + "%" }, function() {
                $this
                  .parent()
                  .prev()
                  .prev()
                  .hide();
              })
              .show("fast", function() {
                config.onMore();
              });
          }
          return false;
        }
      },
      ".morelink"
    );

    return this.each(function() {
      var $this = $(this);

      var content = $this.html();
      var contentlen = $this.text().length;
      if (contentlen > config.showChars + config.minHideChars) {
        var c = content.substr(0, config.showChars);
        if (c.indexOf("<") >= 0) {
          // If there's HTML don't want to cut it
          var inTag = false; // I'm in a tag?
          var bag = ""; // Put the characters to be shown here
          var countChars = 0; // Current bag size
          var openTags = []; // Stack for opened tags, so I can close them later
          var tagName = null;

          for (var i = 0, r = 0; r <= config.showChars; i++) {
            if (content[i] == "<" && !inTag) {
              inTag = true;

              // This could be "tag" or "/tag"
              tagName = content.substring(i + 1, content.indexOf(">", i));

              // If its a closing tag
              if (tagName[0] == "/") {
                if (tagName != "/" + openTags[0]) {
                  config.errMsg =
                    "ERROR en HTML: the top of the stack should be the tag that closes";
                } else {
                  openTags.shift(); // Pops the last tag from the open tag stack (the tag is closed in the retult HTML!)
                }
              } else {
                // There are some nasty tags that don't have a close tag like <br/>
                if (tagName.toLowerCase() != "br") {
                  openTags.unshift(tagName); // Add to start the name of the tag that opens
                }
              }
            }
            if (inTag && content[i] == ">") {
              inTag = false;
            }

            if (inTag) {
              bag += content.charAt(i);
            } else {
              // Add tag name chars to the result
              r++;
              if (countChars <= config.showChars) {
                bag += content.charAt(i); // Fix to ie 7 not allowing you to reference string characters using the []
                countChars++;
              } else {
                // Now I have the characters needed
                if (openTags.length > 0) {
                  // I have unclosed tags
                  //console.log('They were open tags');
                  //console.log(openTags);
                  for (j = 0; j < openTags.length; j++) {
                    //console.log('Cierro tag ' + openTags[j]);
                    bag += "</" + openTags[j] + ">"; // Close all tags that were opened

                    // You could shift the tag from the stack to check if you end with an empty stack, that means you have closed all open tags
                  }
                  break;
                }
              }
            }
          }
          c = $("<div/>")
            .html(
              bag + '<span class="ellip">' + config.ellipsesText + "</span>"
            )
            .html();
        } else {
          c += config.ellipsesText;
        }

        var html =
          '<div class="shortcontent">' +
          c +
          '</div><div class="allcontent">' +
          content +
          '</div><span><a href="javascript://nop/" class="morelink">' +
          config.moreText +
          "</a></span>";

        $this.html(html);
        $this.find(".allcontent").hide(); // Hide all text
        $(".shortcontent p:last", $this).css("margin-bottom", 0); //Remove bottom margin on last paragraph as it's likely shortened
      }
    });
  };
})(jQuery);

$(document).ready(function() {
  //Show our spinny preloader
  document.getElementById("preloader").style.display = "";
  //Gets the data
  $.getJSON(
    "https://archive.org/advancedsearch.php?q=collection%3Aprelinger+avg_rating%3A[4.50+TO+5.00]&fl[]=avg_rating&fl[]=creator&fl[]=description&fl[]=identifier&fl[]=title&fl[]=year&sort[]=&sort[]=&sort[]=&rows=550&page=1&output=json&callback=?",
    function(data) {
      console.log(data);
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

      class PrelingerVideo {
        constructor(theBookStuff) {
          this.theBookStuff = theBookStuff;
        }
        getToAppending() {
          var domsn = document.getElementById("prelinger");
          domsn.insertAdjacentHTML("beforeend", this.theBookStuff);
        }
      }
      for (i = 0; i < ourRandoms.length; i++) {
        let theBookStuff = `<li class=flexy-container><div class="left"><div class="prelinger-pic"><a href="https://archive.org/details/${
          jsonContents[ourRandoms[i]].identifier
        }"><img src="https://archive.org/services/img/${
          jsonContents[ourRandoms[i]].identifier
        }"></a></div></div><div class="right">
        <div class="prelinger-title"><a href="https://archive.org/details/${
          jsonContents[ourRandoms[i]].identifier
        }">${jsonContents[ourRandoms[i]].title}</a></div>
        <div class="prelinger-creator">Creator(s): ${
          jsonContents[ourRandoms[i]].creator
        }</div>
        <div class="prelinger-year">Year: ${
          jsonContents[ourRandoms[i]].year
        }</div>
        <div class="prelinger-rating"><span>User Rating:</span> ${
          jsonContents[ourRandoms[i]].avg_rating
        }</div>
        <div class="prelinger-description">Description: ${
          jsonContents[ourRandoms[i]].description
        }</div>
 
        </div>
        </li>`;

        let ttttt = new PrelingerVideo(theBookStuff);
        ttttt.getToAppending();
      }
      $(".prelinger-description").shorten();
    }
  );
});

{% endhighlight %}
