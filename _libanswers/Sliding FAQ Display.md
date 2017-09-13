---
title: 'Sliding FAQ Display'
layout: implementations
---

## Description
        
This grabs some of your pithy and well-wrought library FAQ entries from LibAnswers and puts them into a sliding display. My example pulls all published FAQs and sorts them in descending order by total FAQ hits. There's other options! You can also retrieve FAQs from only specific topics or those that match a keyword search. They can also be sorted by when it was updated or created (among other options).

The sliding is achieved using [Tiny Carousel](https://baijs.com/tinycarousel) by Maarten Baijs which is open-source and licensed using the MIT license.

### Screenshot

In production the slide is smoother than it appears in the GIF. It lost something in the conversion :)

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}.gif){:.screenshot}

   
### More details
The API call, which is written in jQuery, will send your institution ID to LibAnswers and get a response filled with FAQs (their questions, answers and other information). We pull the question itself and the URL and build a list with them all.
        
We cycle through all the results, pulling the parts we need and assembling them into links for a ```<ul>```. Once these are populated, Tiny Carousel works its magic on the list. We have control over how long entries are shown before sliding to the next. A full list of options and how to manipulate them are available at [Tiny Carousel](https://baijs.com/tinycarousel).
    
## The Code

#### HTML/CSS

{% highlight html linenos %}
  <h3><a href="YOUR_LIBANSWERS_SYSTEM">Library FAQ</a></h3>
   <div id="slider1">
    <div class="viewport">
        <ul id="foofoo" class="overview"></ul>
    </div>
   </div>

{% endhighlight %}

{% highlight css linenos %}

/*For our screenshot*/
h3 {font-family: 'Martel', sans-serif;font-size: 2.5em; letter-spacing: 2px;margin-bottom: 5px;}
h3 a {color: black;text-decoration: none;}
li {font-family: 'Martel', serif;font-size: 1.1em;line-height: 1.2em;}
li a {text-decoration: none;}

/* for Tiny Carousel - Some of these can be changed. Others will break it if modified. */
#slider1 { height: 1%; overflow: hidden; padding: 0 0 10px; }
#slider1 .viewport { float: left; width: 240px; height: 125px; overflow: hidden; position: relative; }
#slider1 .overview { list-style: none; position: absolute; padding: 0; margin: 0; width: 240px; left: 0; top: 0; }
#slider1 .overview li { float: left; margin: 0 20px 0 0; padding: 1px; width: 236px;text-align: left;}

{% endhighlight %}

#### JavaScript/jQuery


{% highlight javascript linenos %}
  $(document).ready(function () {
         $.getJSON('https://api2.libanswers.com/1.0/faqs?iid=000&sort=totalhits&sort_dir=desc&callback=?', function (result) {
             var myResults = result.data.faqs;
             var foofoo = $.map(myResults, function (foomee, index) {
                 var listItem = $('<li></li>');
                 $('<a href=' + foomee.url.public + '>' + foomee.question + '</a>').appendTo(listItem);
                 return listItem;
                 });
             $('#foofoo').html(foofoo)
             $('#slider1').tinycarousel({
                     interval: true,
                     intervalTime: 3000,
                     infinite: true
                 });
             });
     
/*! tinycarousel - v2.1.8 - 2015-11-11
 * https://baijs.com/tinycarousel
 *
 * Copyright (c) 2015 Maarten Baijs <wieringen@gmail.com>;
 * Licensed under the MIT license */

!function(a){"function"==typeof define&&define.amd?define(["jquery"],a):"object"==typeof exports?module.exports=a(require("jquery")):a(jQuery)}(function(a){function b(b,e){function f(){return i.update(),i.move(i.slideCurrent),g(),i}function g(){i.options.buttons&&(n.click(function(){return i.move(--t),!1}),m.click(function(){return i.move(++t),!1})),a(window).resize(i.update),i.options.bullets&&b.on("click",".bullet",function(){return i.move(t=+a(this).attr("data-slide")),!1})}function h(){i.options.buttons&&!i.options.infinite&&(n.toggleClass("disable",i.slideCurrent<=0),m.toggleClass("disable",i.slideCurrent>=i.slidesTotal-r)),i.options.bullets&&(o.removeClass("active"),a(o[i.slideCurrent]).addClass("active"))}this.options=a.extend({},d,e),this._defaults=d,this._name=c;var i=this,j=b.find(".viewport:first"),k=b.find(".overview:first"),l=null,m=b.find(".next:first"),n=b.find(".prev:first"),o=b.find(".bullet"),p=0,q={},r=0,s=0,t=0,u="x"===this.options.axis,v=u?"Width":"Height",w=u?"left":"top",x=null;return this.slideCurrent=0,this.slidesTotal=0,this.intervalActive=!1,this.update=function(){return k.find(".mirrored").remove(),l=k.children(),p=j[0]["offset"+v],s=l.first()["outer"+v](!0),i.slidesTotal=l.length,i.slideCurrent=i.options.start||0,r=Math.ceil(p/s),k.append(l.slice(0,r).clone().addClass("mirrored")),k.css(v.toLowerCase(),s*(i.slidesTotal+r)),h(),i},this.start=function(){return i.options.interval&&(clearTimeout(x),i.intervalActive=!0,x=setTimeout(function(){i.move(++t)},i.options.intervalTime)),i},this.stop=function(){return clearTimeout(x),i.intervalActive=!1,i},this.move=function(a){return t=isNaN(a)?i.slideCurrent:a,i.slideCurrent=t%i.slidesTotal,0>t&&(i.slideCurrent=t=i.slidesTotal-1,k.css(w,-i.slidesTotal*s)),t>i.slidesTotal&&(i.slideCurrent=t=1,k.css(w,0)),q[w]=-t*s,k.animate(q,{queue:!1,duration:i.options.animation?i.options.animationTime:0,always:function(){b.trigger("move",[l[i.slideCurrent],i.slideCurrent])}}),h(),i.start(),i},f()}var c="tinycarousel",d={start:0,axis:"x",buttons:!0,bullets:!1,interval:!1,intervalTime:3e3,animation:!0,animationTime:1e3,infinite:!0};a.fn[c]=function(d){return this.each(function(){a.data(this,"plugin_"+c)||a.data(this,"plugin_"+c,new b(a(this),d))})}});

 }); //end of (document).ready
{% endhighlight %}

* Line 2: ```iid=000``` will have a different number.
* Line 2: ```&sort=totalhits&sort_dir=desc``` are what tells it to sort by total hits, descending. These are from the LibAnswers API documentation available in your LA system. 
* Lines 3-8: Works to grab what we want from the returned JSON data and populates a ```<li>``` with it.
* Line 9: Attaches the resulting set of list items to the HTML ID ```#foofoo```
* Lines 10-13: Our Tiny Carousel options.
* Line 23: All of Tiny Carousel. Take care to keep it after the code which is above it because our JSON items need to populate before Tiny Carousel activates.