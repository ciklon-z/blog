---
layout: post
title: ! 'File scanner web app (Part 5 of 5): Finishing touches'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1390754115'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
<a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-1-of-5-stand-up-and-webserver/">Part 1</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-2-of-5-upload-files/">Part 2</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-3-of-5-yara-signatures/">Part 3</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-4-of-5-scanning-files-from-the-web-app/">Part 4</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-5-of-5-finishing-touches/">Part 5</a>

After completing the previous lessons we now have a full-scale web app where a user can upload a file, get it scanned, and if he knows the right URL's he can see the results.  But it's completely unusable because none of that data get's tied together and presentable for the user.  So it's time to do that and the remaining clean-up.

<h3>Bootstrap</h3>
First, we'll download Twitter's <a href="http://getbootstrap.com/">Bootstrap</a> library which will apply some CSS and make our site not look so ugly.  We'll also get jquery.

<pre>
cd /var/apps/scanner/static/
wget https://github.com/twbs/bootstrap/releases/download/v3.0.0/bootstrap-3.0.0-dist.zip
unzip bootstrap-3.0.0-dist.zip
rm -f bootstrap-3.0.0-dist.zip
mv dist/* .
rm -rf dist
wget -O js/jquery.min.js http://code.jquery.com/jquery-2.0.3.min.js
wget -O js/jquery-2.0.3.min.map http://code.jquery.com/jquery-2.0.3.min.map
</pre>

Now we need to make our site use these by editing our <tt>static/index.htm</tt> file.
First, I'm going to clean up our .htm file a bit by putting some of the code into additional files.  We'll put the following into <tt>js/scanner.js</tt>
{% highlight javascript %}
Dropzone.options.mydropzone = {
  previewTemplate : '<div class="preview file-preview">\
  <div class="dz-details">\
    <b class="dz-filename"><span data-dz-name></span></b>\
    <b class="dz-size" data-dz-size></b>\
  </div>\
  <div class="dz-progress"><span class="dz-upload" data-dz-uploadprogress></span></div>\
  <div class="dz-error-message"><span data-dz-errormessage></span></div>',
  init: function() {
    this.on("complete", function(file) { console.log("Upload complete"); });
  }
};
{% endhighlight %}

and put the following into <tt>css/scanner.css</tt>
{% highlight css %}
.dropzone {
    border-style:dotted; 
    border-width:2px;
    min-height: 100px;
    height:100px;
}
{% endhighlight %}

We'll also move <tt>dropzone.js</tt> to the <tt>js</tt> directory.
<pre>
mv dropzone.js js/.
</pre>

Now we set our <tt>index.htm</tt> to look like the following so it uses these changes and also uses bootstrap:
{% highlight html %}
<html>
<head>
    <link href="css/bootstrap.min.css" rel="stylesheet" media="screen">
    <link href="css/scanner.css" rel="stylesheet">
</head>

<body>
    <div class="container">
    <h1>File Scanner</h1>

    <form action="/file-upload"
          class="dropzone"
          id="mydropzone"></form>
    </div>
</body>

<!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
<script src="js/jquery.min.js"></script>
<!-- Include all compiled plugins (below), or include individual files as needed -->
<script src="js/bootstrap.min.js"></script>

<script src="js/dropzone.js"></script>
<script src="js/scanner.js"></script>
{% endhighlight %}

Easy.  Now we're using bootstrap.  The only difference you should see is that the font has changed and you have a margin around the content.

<h3>DatatablesJS</h3>
Time to display our database data.  We'll use <a href="http://datatables.net/index">DatatablesJS</a>.
<pre>
wget http://datatables.net/releases/DataTables-1.9.4.zip
unzip DataTables-1.9.4.zip
rm -f DataTables-1.9.4.zip
mv DataTables-1.9.4/media/js/jquery.dataTables.min.js js/.
mv DataTables-1.9.4/media/css/jquery.dataTables.css css/.
mv DataTables-1.9.4/media/images .
</pre>

Integrating DatatablesJS into our <tt>index.htm</tt> is a little painful the first time you do it.  What happens is we create an initial mock-up of our table in the HTML, then in the <tt>scanner.js</tt> we tell it to pull in our data via ajax and redo our table using that.  I then added in code so I can click on rows and show the rule matches at the bottom (again via more javascript).  Then if you click on a rule that matched it takes you to the actual YARA signature code that is stored in the database.

First the <tt>index.htm</tt> file.

(Highlight lines:5,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,42)

{% highlight html linenos=table %}
<html>
<head>
    <link href="css/bootstrap.min.css" rel="stylesheet" media="screen">
    <link href="css/scanner.css" rel="stylesheet">
    <link href="css/jquery.dataTables.css" rel="stylesheet">
</head>

<body>
    <div class="container">
    <h1>File Scanner</h1>

    <form action="/file-upload"
          class="dropzone"
          id="mydropzone"></form>

    <table cellpadding="0" cellspacing="0" border="0" class="display" id="datatable">
        <thead>
        <tr>
            <th>ID</th>
            <th>Submission Date</th>
            <th>Filename</th>
            <th>Filesize</th>
            <th>MD5</th>
        </tr>
        </thead>
        <tbody></tbody>
    </table>

    <hr>
    <h3>Match data</h3>
    <div id="matchData">Click an element to display it's matches</div>

    </div>
</body>

<!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
<script src="js/jquery.min.js"></script>
<!-- Include all compiled plugins (below), or include individual files as needed -->
<script src="js/bootstrap.min.js"></script>

<script src="js/dropzone.js"></script>
<script src="js/jquery.dataTables.min.js"></script>
<script src="js/scanner.js"></script>

{% endhighlight %}

Now a small change to our <tt>scanner.css</tt>.

(Highlight lines: 8)

{% highlight css linenos=table %}
.dropzone {
    border-style:dotted; 
    border-width:2px;
    min-height: 100px;
    height:100px;
}

tr.row_selected td {background-color:#ffff00}
{% endhighlight %}

Our <tt>scanner.js</tt> requires a large addition.

(Highlight lines: 15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44)

{% highlight javascript linenos=table %}
Dropzone.options.mydropzone = {
    previewTemplate : '<div class="preview file-preview">\
    <div class="dz-details">\
        <b class="dz-filename"><span data-dz-name></span></b>\
        <b class="dz-size" data-dz-size></b>\
    </div>\
    <div class="dz-progress"><span class="dz-upload" data-dz-uploadprogress></span></div>\
    <div class="dz-error-message"><span data-dz-errormessage></span></div>\
    </div>',
    init: function() {
        this.on("complete", function(file) { console.log("Upload complete"); });
    }
};


$(document).ready(function() {
    oTable = $('#datatable').dataTable( {
        "bProcessing": true,
        "sAjaxDataProp": '',
        "sAjaxSource": '/getFiles',
    } );

    $("#datatable tbody").click(function(event){
            $(oTable.fnSettings().aoData).each(function (){
                $(this.nTr).removeClass('row_selected');
            });
            $(event.target.parentNode).addClass('row_selected');

            var id = oTable.fnGetData(event.target.parentNode)[0];
            $.ajax({
              url: "/getMatches/"+id,
            }).done(function(matches) {
                content = "";
                matches = JSON.parse(matches);
                for (var i in matches) {
                  match = matches[i];
                  content += "Match: <a href='/getRule/"+match['rule_id']+"'>"+match['description']+"</a><br>"
                }
                if (content == "") {
                    content = "No matches found";
                }
                $("#matchData").html(content);
            });
    });
} );
{% endhighlight %}

At this point if you visit the web app in the browser, drop some files and click on the rows, you should be able to see something like the following:
<a href="http://0xdabbad00.com/wp-content/uploads/2013/09/Screen-Shot-2013-09-02-at-3.30.33-PM.png"><img src="http://0xdabbad00.com/wp-content/uploads/2013/09/Screen-Shot-2013-09-02-at-3.30.33-PM-300x152.png" alt="" title="Finished app" width="300" height="152" class="aligncenter size-medium wp-image-1190" /></a>

Now we have a functional web app!  It's still missing a lot of UI work (like refreshing our table as we drop files to it), but its useful.  The only final step is to make our two python scripts into services, so we can shutdown the VM and restart it and use our webapp immediately without having to SSH in and start up some background processes by hand.  For that we'll use <a href="http://supervisord.org/">supervisord</a>.

<h3>Supervisord</h3>
Install supervisord first.
<pre>
sudo apt-get install supervisor
</pre>

Now we need to create a config file telling supervisord to run our two services.  Create a file called <tt>/etc/supervisor/conf.d/scanner.conf</tt> with the following contents:
<pre>
[program:webserver]
command=python webserver.py
directory=/var/apps/scanner

[program:yarascanner]
command=python yarascanner.py
directory=/var/apps/scanner
</pre>

Now kill any current processes you have for running the webserver or yarascanner and tell supervisord to load these processes
<pre>
sudo supervisorctld reload
</pre>

That's it, we're done!  You can now reboot the VM and when it comes back up your server will be ready for you.

<h3>Conclusion</h3>
That concludes this training series.  You now have a working web app.  It's not great: You could add a lot to it.  But now you know how to use a lot of different tools to make something usable.  You know how to tie different processes together.  These are important skills in today's development environment.

If you messed up along the way you can check out the completed app on github at: <a href="https://github.com/0xdabbad00/filescannner_webapp">https://github.com/0xdabbad00/filescannner_webapp</a>  

Now go write some YARA signatures!

Note that this web app probably should not be exposed to the Internet at large, as it's file upload could DoS your app pretty quickly.

Please let me know if this was helpful, if the format was confusing, etc.  Seriously, any feedback is welcome.  I'm hoping to improve your ability to develop software.  You can find me as <a href="https://twitter.com/0xdabbad00">@0xdabbad00</a> or email me at 0xdabbad00 on gmail.
