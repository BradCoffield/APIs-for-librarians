---
layout: implementations
title: "RSS Feed Creation for Repository Items"
author: "Andrew Lechlak"
author_email: "Andy.Lechlak@toledolibrary.org" 
tags: archives, zapier, rss, csv, php, bitly
---

{:.implementation-author}
This page courtesy of: [{{page.author}}](https://github.com/Lechlak)

## Description

The goal was to create an RSS feed from CONTENTdm using a CSV list of images that were timed based on popular cultural events. I.E. Hispanic Heritage month, New Years Day, Spring, etc. So why use a CSV? Well a database requires someone with IT knowledge to build. A CSV is created inside a basic spreadsheet software program like Excel. If we can build the application so it is accessible to more people, then more people can create and iterate on this idea for their collections.

We used the [ContentDM APIs](https://www.oclc.org/support/services/contentdm/help/customizing-website-help/other-customizations/contentdm-api-reference.en.html) and [IIIF (International Image Interoperability Framework)](https://iiif.io/) for building our RSS Feed. We were then able to leverage our RSS feed to work with [Zapier](https://www.zapier.com) to automatically share items to various social media platforms! 



### Screenshot

![{{page.title}} screenshot]({{site.baseurl}}/assets/{{page.title}}-screenshot.png){:.screenshot}

### More details

This guide focuses on using the ContentDM API with a spreadsheet to enable less technical users to maintain collections. For more information, and how to work with Zapier to automate item promotion, please see the [complete instructions](http://www.opentoledo.org/?p=353) from Open Toledo.

## The Code

#### CSV

{:.code-notes}

* While it is not really code, the format of the CSV was important to the actual coding of the project.
  {% highlight csv linenos %}
  Date,CDM# MD,CDM# Image,Notes
  10/4/2018,304,304,
  10/5/2018,12642,12641,
  10/6/2018,12644,12643{% endhighlight %}

#### PHP

##### Notes for implementation:

{:.code-notes}

* Pay attention to lines 3, 6, 7, 8, 11, and 12. You will need to change values for them.
* Step-by-step instructions are available at [Open Toledo](http://www.opentoledo.org/?p=353)


##### The code itself:

{% highlight php linenos %} <?php
//csv file
$csvfile = "ImageShareSchedule.csv"; //change this

//cdm data
$cdmcollection = "p16007coll33"; //change this
$cdmserver = "server16007"; //change this
$basecdmurl = "http://ohiomemory.org"; //change this - probably too https://$cdmserver.contentdm.oclc.org/ - you can view this by navigating to your collection and looking at the URL

//bit.ly
$login = "bitly-login"; //change this
$appkey = "bitly-app-key"; //change this
$format = "txt";

function get_bitly_short_url($url, $login, $appkey, $format = 'txt')
{
    $connectURL = 'http://api.bit.ly/v3/shorten?login=' . $login . '&apiKey=' . $appkey . '&uri=' . urlencode($url) . '&format=' . $format;
    //echo $connectURL;

    $ch2 = curl_init();
    curl_setopt($ch2, CURLOPT_SSL_VERIFYPEER, false);
    curl_setopt($ch2, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch2, CURLOPT_URL, $connectURL);
    $result = curl_exec($ch2);
    curl_close($ch2);

    $obj = json_decode($result, true);
    //echo $result;
    return $result;

}

$today = getdate();
$d = $today['mday'];
$m = $today['mon'];
$y = $today['year'];
$currentdate = "$m/$d/$y";
//echo $currentdate;
// echo "<br/>";

//$row = 4;
//$column = 2;
//echo $csv[$row][$column];

$csv = array();
$x = 0;

if (($handle = fopen($csvfile, "r")) !== false) {
    while (($data = fgetcsv($handle, 1000, ",")) !== false) {
        $csv[] = $data;
        $rdate = $csv[$x][0];
        //echo "x = $x | date = $rdate";
        //echo "</br>";

        if ($currentdate == $rdate) {
            //echo "x = $x | date = $rdate";
            //echo "</br>";
            $cdmmdpointer = $csv[$x][1];
            $cdmimgpointer = $csv[$x][2];
            $cdmimg = "$basecdmurl/digital/iiif/$cdmcollection/$cdmimgpointer/full/400,200/0/default.jpg";
            $cdmmd = "https://$cdmserver.contentdm.oclc.org/dmwebservices/index.php?q=dmGetItemInfo/$cdmcollection/$cdmmdpointer/json";

            $ch2 = curl_init();
            curl_setopt($ch2, CURLOPT_SSL_VERIFYPEER, false);
            curl_setopt($ch2, CURLOPT_RETURNTRANSFER, true);
            curl_setopt($ch2, CURLOPT_URL, "https://$cdmserver.contentdm.oclc.org/dmwebservices/index.php?q=dmGetItemInfo/$cdmcollection/$cdmmdpointer/json");
            $result2 = curl_exec($ch2);
            curl_close($ch2);

            $obj2 = json_decode($result2, true);
//echo $ojb2;
            $title = $obj2['title'];
//echo $title;
            //$title = mysql_real_escape_string($title);

            $cdmlink = "$basecdmurl/cdm/singleitem/collection/$cdmcollection/id/$cdmimgpointer/rec/1";

            //Keep bit.ly
            $bitlycdmlink = get_bitly_short_url($cdmlink, $login, $appkey);

            //Remove bit.ly
            //$bitlycdmlink = $cdmlink;

//RSS FEED CREATION
            echo '<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
<channel>
<title>' . $title . '</title>
<link>' . $bitlycdmlink . '</link>
<description>' . $cdmimg . '</description>
<item>
<title>' . $title . '</title>
<link>' . $bitlycdmlink . '</link>
<description>' . $cdmimg . '</description>
</item>
</channel>
</rss>';

        }
        $x++;
    }
}

fclose($handle);

{% endhighlight %}
