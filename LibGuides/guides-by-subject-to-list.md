# LibGuides, by Subject, to an Unordered List

## Description

The LibGuides API allows you to limit the API's response to return only guides of a certain subject. Subjects are defined inside of LG, as assigned by admins or guide authors to individual guides.

Two nice implementations of this idea exist at [Rochester Institute of Technology's LG homepage](http://infoguides.rit.edu/) and at my library, [SFU Library](library.francis.edu) (under the heading 'Research Guides') My library's example is actually using the LG 'widget' which you can build from within the LG interface but the basic results are the same. Use the API call I'm detailing here if you want to end up with more control over syling your list.

### More details

The API call, which is written in jQuery, will send your authorization key along with some pieces of information that will tell LG to only return JSON data about guides from a single subject or subjects.

Once that data is returned we pull out the individual title of a guide and its URL and place those pieces into an LI element. The jQuery will then enumerate through each JSON entry doing the same thing again and again until complete. You might have a list of 3 items or 300, it doesn't matter.

You link up the jQuery and your actual HTML using a specific ID name in your HTML, assigned to a UL. If you wanted a variety of different such lists you would need to use that many different pieces of this code, with your subject identifiers in the JSON call and the HTML/JS linkup changed.

### The Code

#### HTML

```html
<ul id="business-guides"></ul>
```
Simple, right?

#### JavaScript/jQuery

```javascript

$.getJSON('https://lgapi-us.libapps.com/1.1/guides?site_id=foo1&key=foo2&status=1&subject_ids=38607', function (result) {

    var businessGuides = $.map(result, function (guidesData, index) {
        var listItem = $('<li></li>');
        $('<a href=/"' + guidesData.url + '/">' + guidesData.name + '</a>').appendTo(listItem);
        return listItem;

    });
    // this part with the # is what is linking up to the HTML on your page - so the JS knows where to put your list of guides.
    $('#business-guides').html(businessGuides);
});
```
**Comments on the code:**

* foo1 and foo2 are placeholders. Fill these in with the information provided by Springshare.
* ```
status=1
``` refers to published guides.
* ```
subject_ids=38607
``` refers to the specific code for the subject 'business' in our system. You'll need to replace that number.
* As noted inside the code you'll need to adjust the ```
#business-guides
``` as appropriate for your site's code.


