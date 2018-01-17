---
layout: implementations
title: "All Texts by Keyword"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com" 
tags: text-collections
---

## Description

This asks the [archive.org](https://www.archive.org/) API for a list of all of
the texts (books, magazines, etc.) returned from a keyword search (that you
specify) and then selects two from that list for display. It will select a new
set on each page load.

Something like this can help promote seredipitous discovery in your research
guides. It also has the benefit of being something that is fresh and constantly
changing without you needing to constantly update it.

### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}

### More details

It would be easy to adjust the number of entries displayed down to one and also
possible, but not quite easy, to adjust them as high as you would want.

Overly long item descriptions are being hidden behind a link ("read more"). When
clicked that will expand and show the rest of the description. I decided to do
this because some of the descriptions were really long and some were quite brief
and it the mixture of them looked bad. You can not do this if you choose, or you
can modify the number of characters that are permitted to be displayed.

## The Code

#### HTML

{:.code-notes}

* Just the empty `<ul>` our JS will work with. 
{% highlight html linenos %}
 
  <ul id="IA-books"></ul> {% endhighlight %}

#### CSS

{:.code-notes}

* This CSS just uses flexbox to position the image to the left and the text to
  the right.
* The media query makes it so that on small screens the text switches to below
  the image.

{% highlight css linenos %} #IA-books { list-style-type: none; }
.flexy-container{ display: flex; align-items: center; } .flexy-container .left {
padding: 1rem; } @media (max-width: 500px) { .flexy-container{flex-wrap: wrap;}
.IA-book-image img {display: block;margin: 0 auto;} } a {text-decoration: dotted
underline gray;color:#333;}

{% endhighlight %}

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}

* Lines 21-181: This part of code is what does the description hiding and is
  from [http://viralpatel.net](http://viralpatel.net).
* Line 192: This is where we change the topic of our keyword search. Look for
  `q=%22beat+generation%22`. What's happening here is that `%22` represents
  quotation marks. So, if you want to search for a phrase you'll need to use
  these. But you don't have to. `+` signs have to go where spaces would be. So, instead of what's there you could put `q=alien+jellybeans` and that would work as expected, not searching for them as a phrase.

##### The code itself:

{% highlight javascript linenos %}

$(document).ready(function () {
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

    (function ($) {
        $.fn.shorten = function (settings) {
            "use strict";

            var config = {
                showChars: 250,
                minHideChars: 10,
                ellipsesText: "...",
                moreText: "Read more",
                lessText: "Read less",
                onLess: function () {},
                onMore: function () {},
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

            $(document).on({
                    click: function () {
                        var $this = $(this);
                        if ($this.hasClass("less")) {
                            $this.removeClass("less");
                            $this.html(config.moreText);
                            $this
                                .parent()
                                .prev()
                                .animate({
                                    height: "0" + "%"
                                }, function () {
                                    $this
                                        .parent()
                                        .prev()
                                        .prev()
                                        .show();
                                })
                                .hide("fast", function () {
                                    config.onLess();
                                });
                        } else {
                            $this.addClass("less");
                            $this.html(config.lessText);
                            $this
                                .parent()
                                .prev()
                                .animate({
                                    height: "100" + "%"
                                }, function () {
                                    $this
                                        .parent()
                                        .prev()
                                        .prev()
                                        .hide();
                                })
                                .show("fast", function () {
                                    config.onMore();
                                });
                        }
                        return false;
                    }
                },
                ".morelink"
            );

            return this.each(function () {
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
    // Shows our spinner
    //$("#preloader").show();



    $.getJSON("https://archive.org/advancedsearch.php?q=%22beat+generation%22++mediatype%3Atexts&fl%5B%5D=creator&fl%5B%5D=description&fl%5B%5D=identifier&fl%5B%5D=publisher&fl%5B%5D=title&sort%5B%5D=&sort%5B%5D=&sort%5B%5D=&rows=500&page=1&output=json&callback=?", function (data) {
        // Removes our spinner once the ajax request is being displayed
       // $("#preloader").hide();



        //Get two random non-identical numbers
        let num1, num2;
        const getTwoRandom = function () {
            num1 = getRandomIntInclusive(0, data.response.docs.length);
            num2 = getRandomIntInclusive(0, data.response.docs.length);

            if (num1 == num2) {
                getTwoRandom();
            }
        };
        getTwoRandom(data);

        //Let's place our books on the page

        $("#IA-books").append(`<li class=flexy-container>
    <div class="left"><div class="IA-book-image"><a href="https://archive.org/details/${data.response.docs[num1].identifier}"><img src="https://archive.org/services/img/${
      data.response.docs[num1].identifier
    }"></a></div></div>

    <div class="right"><div class="IA-book-title"><h4> <a href="https://archive.org/details/${data.response.docs[num1].identifier}"> ${data.response.docs[num1].title}</a></h4>
    <h4>Description:</h4><p class="IA-book-description">${data.response.docs[num1].description}</p>
    </div></div>
    </li>`);

        $("#IA-books").append(`<li class=flexy-container>
    <div class="left"><div class="IA-book-image"><a href="https://archive.org/details/${data.response.docs[num1].identifier}"><img src="https://archive.org/services/img/${
      data.response.docs[num2].identifier
    }"></a></div></div>

    <div class="right"><div class="IA-book-title"><h4> <a href="https://archive.org/details/${data.response.docs[num1].identifier}"> ${data.response.docs[num2].title}</a></h4>
    <h4>Description:</h4><p class="IA-book-description">${data.response.docs[num2].description}</p>
    </div></div>
    </li>`);

 
        $(".IA-book-description").shorten();
    });
});
  {% endhighlight %}
