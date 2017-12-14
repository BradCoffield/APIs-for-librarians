---
layout: implementations
title: "Chat Status"
author: "Brad Coffield"
author_email: "bcoffield@gmail.com"
---
## Description
        
This checks the status of one of your LibChat widgets and based on a true/false response displays custom text. So, if the widget is online (meaning that a particular chat queue is active) you can have it say whatever you want. In my example I have ```Online!``` and ```Offline```, respectively.

### Screenshot

The styling here is more just a proof-of-concept. But you have the full range of CSS and JS at your disposal to make it look as you want.

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.jpg){:.screenshot}
![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot2.jpg){:.screenshot}

       
### More details
The API call, which is written in jQuery, will send a widget ID to your LibChat system and get a response as to whether or not that particular widget is online. You can use this to monitor general chat queues or the chat queues automatically created for each LibChat librarian.
        
The code is just a simple if/else that checks to see if LibChat has responded ```true```. If so, you can have it display one thing and if not then another. The success response can be linked to a pop-out full chat interface and the fail response can be linked to your LibAnswers system. Or they can not be linked. Or linked to anything you'd like.

SpringShare does have a button widget that does essentially the same thing as this and, in fact, they do allow heavy customization of it. Though doing it this way gives you complete control over it.
    
## The Code

#### HTML/CSS

{% highlight html linenos %}
<div id="libchat-status-container">Chat with a Librarian! 
    <span id="chat-status"></span>
</div>

{% endhighlight %}

{% highlight css linenos %}
#libchat-status-container {
    text-align: center;
    font-size: 1.2em;
}
#chat-online, #chat-offline {
    font-weight: bold;
    font-size:2em;
}
#chat-online {
    color: green;
}
#chat-offline {
    color: red;
}

{% endhighlight %}

#### JavaScript/jQuery

##### Notes for implementation:

{:.code-notes}
* Line 2: ```1234``` is where you put the widget ID.
* Lines 4 and 6: The stuff in paren after ```.html``` is what is getting injected into your html. This is where you can customize the displayed text, add or remove linkage etc. 

##### The code itself:
{% highlight javascript linenos %}
 $(document).ready(function () {
    $.getJSON("https://api2.libanswers.com/1.0/chat/widgets/status/iid=1234", function (json) {
        if (json.online === true) {
            $('#chat-status').html('<br/><p id="chat-online"><a href="PUT_YOUR_CHAT_LINK_URL_HERE">Online!</a></p>');
        } else {
            $('#chat-status').html('<br/><span id="chat-offline">Offline</span>');
        };
    })
});{% endhighlight %}

