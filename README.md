# PubSubHubbub Subscribing for CouchDB

Make CouchDB accept and store PubSubHubbub feeds! Original concept by @rdfturtle (Tyler Gillies)

# Why is this cool?

So I reckon you have some old and busted RSS or ATOM feeds that you need to pull from whenever you want updates, right?

Well, little buddy, it's time to enter the world of push!

Hook your pull feeds into the magic that is PubSubHubbubb (superfeedr.com does this for you... _for free!_) and they will push to a specific PubSubHubbubb subscriber endpoint whenever they have new content.

You might be asking "where am I going to get a valid PubSubHubbubb subscriber endpoint to store all of these awesome feed pushes"? Well, i'm glad you asked!

With Couchpubtato, you can make any Couch database act like a valid PubSubHubbubb subscriber endpoint.

# I still don't get it

Ok, so lets say you want to embed a feed of upcoming calendar events on your _sweet blog_, but the calendar page only has a junky RSS feed! You can use Superfeedr to push new RSS updates in realtime to a CouchDB of your choice, which turns your previously junky RSS feed into a full JSONP enabled API for maximum client side widget goodness!

Here's an example of this whole thingy working: I log all Portland, OR 911 calls and make them available as ActivityStreams here: <code>http://pdxapi.com/pdx911/feed?descending=true&limit=5</code>

# ActivityStreams

By default this will convert any incoming XML RSS/ATOM feed data into JSON [ActivityStreams](http://activitystrea.ms) format. ActivityStreams are the new hotness, pure XML ATOM/RSS is old and busted. Here's an example of an ActivityStreams formatted feed item:

    {
       "postedTime":"2010-10-14T00:58:32Z",
       "object": {
           "permalinkUrl": "http://rss.cnn.com/~r/rss/cnn_latest/~3/s52R1lImWu0/index.html",
           "objectType": "article",
           "summary": "O'Donnell, Coons stage feisty debate in Delaware"
       },
       "verb": "post",
       "actor": {
           "permalinkUrl": "http://rss.cnn.com/~r/rss/cnn_latest/~3/s52R1lImWu0/index.html",
           "objectType": "service",
           "displayName": "CNN.com Recently Published/Updated"
       }
    }

# Quick install

* Get a free hosted Couch from [CouchOne](http://couchone.com/get) and a free Superfeedr subscriber account from [Superfeedr](http://superfeedr.com)
* Make a new database on your couch: <code>curl -X PUT http://YOURCOUCH/DBNAME</code>
* Copy Couchpubtato to the new db: <code>curl -X POST http://YOURCOUCH/\_replicate -d '{"source":"http://max.couchone.com/apps","target":"apps", "doc\_ids":["_design/push"]}'</code>
* Tell Superfeedr to store an XML feed in your Couch: <code>curl -X POST http://superfeedr.com/hubbub -u'SUPERFEEDRUSERNAME:SUPERFEEDRPASSWORD' -d'hub.mode=subscribe' -d'hub.verify=sync' -d'hub.topic=YOURFEEDURL' -d'hub.callback=http://YOURCOUCH/DBNAME/\_design/push/_rewrite/xml' -D-</code>

# In-depth install

You can use any CouchDB hosted pubicly (so that Superfeedr can post updates to it), but I'll assume you're working with a free Couch hosted by [CouchOne](http://couchone.com/get). You'll have to create a new database to store your data. I'll call mine <code>awesome-events</code>. You can do this from http://YOURCOUCH/_utils.

You can either replicate the couchapp from my couch [max.couchone.com/apps/_design/push](http://max.couchone.com/apps/_design/push) (quickest option) or, if you want to hack on the Couchpubtato source code first, you'll need to install the [CouchApp command line utility](http://couchapp.org/page/installing) and check out this repo.

If you want to hack on Couchpubtato/build it yourself, once you have the couchapp utility working, <code>git clone</code> this repo and go into this folder and execute <code>couchapp init</code>. To upload Couchpubtato into your couch just run <code>couchapp push http://YOURCOUCH/DATABASENAME</code>. Otherwise see the Quick install section above.

When you push Couchpubtato into your Couch it will enhance your new database with the magical PubSubHubbubb sprinkles contained in Couchpubtato and teach your database how to store PubSubHubbubb data.

Once your database is ready to store data, we need to use Superfeedr to hook up a feed to dump into your new PubSubHubbubb enabled Couch database.

Now go get a free Subscriber account at [Superfeedr](http://superfeedr.com)

To subscribe to a feed, use the following <code>curl</code> command:

<code>curl -X POST http://superfeedr.com/hubbub -u'SUPERFEEDRUSERNAME:SUPERFEEDRPASSWORD' -d'hub.mode=subscribe' -d'hub.verify=sync' -d'hub.topic=YOURFEEDURL' -d'hub.callback=http://YOURCOUCH/DATABASENAME/\_design/push/_rewrite/xml' -D-</code>

It should return a <code>204</code> if everything was fine, and if not, it will indicate what was wrong in the BODY. If you don't like curl, there also exist PubSubHubbub libraries in many languages.

Now any feed updates will get sent to your Couch and will be converted and saved as JSON ActivityStreams objects.

# Known issues

Superfeedr will send multiple feed updates in a single update, so if 3 new items were posted to your feed, it will send a single XML update containing 3 items. Couch currently can only create a single document for each update, so you may see documents that contain multiple feed items. A more desirable situation would be a 1:1 ratio of documents in Couch to feed items.

# License

The MIT License

Copyright (c) 2010 Max Ogden and Tyler Gillies

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.