---
layout: implementations
title: "Three Random Naropa Entries"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com" 
tags: naropa
---
## Description
        
This asks the [archive.org](https://www.archive.org/) API for a list of all of the [Naropa University Archive Project's](https://archive.org/details/naropa&tab=about) contents and then selects three from that list for display. It will select a new set on each page load. The Naropa archive features poetry readings and lectures from many prominent Beat (literary movement) figures like Allen Ginsburg and Anne Waldman. There's a lot of great content! 

This implementation would be great for a libguide page on the Beat literary movement.

### Screenshot

This is a screenshot of it inside our libguides. 

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}
       
### More details

It would be easy to adjust the number of entries displayed either up or down. There is a description below the entries that comes from the larger description page at the Archive and which isn't necessary for the entries to work. 

Overly long descriptions are being hidden behind a link ("read more"). When clicked that will expand and show the rest of the description. I decided to do this because some of the descriptions were really long and some were quite brief and it the mixture of them looked bad. You can not do this if you choose, or you can modify the number of characters that are permitted to be displayed.

    
## The Code

#### HTML

{:.code-notes}
* The HTML uses Bootstrap grid syntax to achieve the columns. LibGuides 2.0 is built on Bootstrap so you can paste this into the HTML/Rich Text area (code view) no problem.
* Line 5: I downloaded an animated gif spinner that will display while the API is working and before the results have loaded. You don't have to do this but it does give a visual cue to the user that something is happening behind the scenes.
{% highlight html linenos %}
  <div class="row" id="naropa-row">
    <h3>The Naropa Poetics Audio Archive</h3>
    <p>Three random pieces from the archive. Want more? Reload this page or visit them at <a href="https://archive.org/details/naropa">https://archive.org/details/naropa</a> </p>
    <!-- A spinner animated gif for while the ajax loads --><img src="/assets/img/Eclipse.gif" alt="" id="preloader" style="display:block;margin:0 auto;">
    <!-- Our three random items -->
    <div class="col-md-4">
        <ul id="naropa-1"></ul>
    </div>
    <div class="col-md-4">
        <ul id="naropa-2"></ul>
    </div>
    <div class="col-md-4">
        <ul id="naropa-3"></ul>
    </div>
</div>
<p id="naropa-description">The Naropa University Archive Project is preserving and providing access to over 5000 hours of recordings made at Naropa University in Boulder, Colorado. The library was developed under the auspices of the Jack Kerouac School of Disembodied Poetics (the university's Department of Writing and Poetics) founded in 1974 by poets Anne Waldman and Allen Ginsberg. It contains readings, lectures, performances, seminars, panels and workshops conducted at Naropa by many of the leading figures of the U.S.literary avant-garde. <a href="https://archive.org/details/naropa&tab=about">More info...</a> </p>
</div>

{% endhighlight %}

#### CSS

{:.code-notes}
* The screenshot above is also using Bootstrap CSS for the grid.

{% highlight css linenos %}
#naropa-row h3 + p {
  margin-bottom: 2em;
  margin-top: -0.1em;
}
#naropa-row ul {
  list-style-type: none;
  padding-left: 0px;
}

.naropa-item-title a {
  font-size: 1.2em;
  margin-left: 0;
}
#naropa-row ul li {
  text-align: center;
  margin-bottom: 1em;
}
#naropa-description {
  margin-top: 1em;
}

.morelink {
  font-size: 12px;
  text-transform: lowercase;
}
{% endhighlight %}

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}
* Lines 20-180: This part of code is what does the description hiding and is from [http://viralpatel.net](http://viralpatel.net).  
* Lines 196-205: Could be removed if you only want to display one result. If so then just add `let num1= getRandomIntInclusive(0, data.response.docs.length);` in their stead. 
* Lines 214-227: Also delete these if you only want one result. 

##### The code itself:
{% highlight javascript linenos %}
$(document).ready(function() {
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
        showChars: 250,
        minHideChars: 10,
        ellipsesText: "...",
        moreText: "Read more",
        lessText: "Read less",
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

  //returns a random number from a range
  function getRandomIntInclusive(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }
  //allows our spinner to be seen
  $("#preloader").show();
  $.getJSON(
    "https://archive.org/advancedsearch.php?q=collection%3Anaropa+mediatype%3Aaudio++&fl[]=creator&fl[]=description&fl[]=identifier&fl[]=title&fl[]=year&sort[]=&sort[]=&sort[]=&rows=1000&page=1&output=json&callback=?",
    function(data) {
      //removes our spinner when the ajax request is being displayed
      $("#preloader").hide();
      //selects three different random numbers from within our results set amount and ensures none of them match each other
      let num1, num2, num3;
      let getThreeRandom = function() {
        num1 = getRandomIntInclusive(0, data.response.docs.length);
        num2 = getRandomIntInclusive(0, data.response.docs.length);
        num3 = getRandomIntInclusive(0, data.response.docs.length);
        if (num1 == num2 || num2 == num3 || num1 == num3) {
          getThreeRandom();
        }
      };
      getThreeRandom(data);
      //these use our random numbers to select pieces from our data set and display them on our webpage. Uses font-awesome icon fonts
      $("#naropa-1")
        .append(`<li class="naropa-item-title"><a href="https://archive.org/details/${data
        .response.docs[num1].identifier}"> ${data.response.docs[num1]
        .title}</a></li>
      <li class="naropa-description">${data.response.docs[num1]
        .description}</li>
`);
      $("#naropa-2")
        .append(`<li class="naropa-item-title"><a href="https://archive.org/details/${data
        .response.docs[num2].identifier}"> ${data.response.docs[num2]
        .title}</a></li>
      <li class="naropa-description">${data.response.docs[num2]
        .description}</li>
`);
      $("#naropa-3")
        .append(`<li class="naropa-item-title"><a href="https://archive.org/details/${data
        .response.docs[num3].identifier}"> ${data.response.docs[num3]
        .title}</a></li>
      <li class="naropa-description">${data.response.docs[num3]
        .description}</li>
`);
      $(".naropa-description").shorten();
    }
  );
});
{% endhighlight %}

