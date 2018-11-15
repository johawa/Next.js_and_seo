# This is my Tutorial on how to improve SEO on a Next.js App

Since Next.js Apps are Serveside renderes The Google CrawlBot got issus with catching the all the Data, your App provides.
Next.js uses a spezial Method call getPropsOnInitialLoad, this method provides a Component, data before the Initial Load. The data catched, wil provide the Component with data, to render the whole Content on the Server.

Problem is: Google Crawl Bot cannot catch this data (even it is very clever), this is a huge Problem when it comes to SEO

But you can provide thr Crawl Bot with an Sitemap.xml and robots.txt like every other Website.
For this we are using [a third Party Package]('https://github.com/ekalinin/sitemap.js')

Inside the sitemap.xml we specify which routes to indes (ulr), the change Freqency (changefreg) and the Priority of this Page (priority)

### This example is taken from the offical Docs:

```javascript
var sm = require("sitemap");
// Creates a sitemap object given the input configuration with URLs
var sitemap = sm.createSitemap({ options });
// Generates XML with a callback function
sitemap.toXML(function(err, xml) {
  if (!err) {
    console.log(xml);
  }
});
// Gives you a string containing the XML data
var xml = sitemap.toString();
```

### since we are using [Express]('https://www.npmjs.com/package/express') as our Sever:

```javascript
var express = require("express"),
  sm = require("sitemap");

var app = express(),
  sitemap = sm.createSitemap({
    hostname: "http://example.com",
    cacheTime: 600000, // 600 sec - cache purge period
    urls: [
      { url: "/page-1/", changefreq: "daily", priority: 0.3 },
      { url: "/page-2/", changefreq: "monthly", priority: 0.7 },
      { url: "/page-3/" }, // changefreq: 'weekly',  priority: 0.5
      { url: "/page-4/", img: "http://urlTest.com" }
    ]
  });

app.get("/sitemap.xml", function(req, res) {
  sitemap.toXML(function(err, xml) {
    if (err) {
      return res.status(500).end();
    }
    res.header("Content-Type", "application/xml");
    res.send(xml);
  });
});

app.listen(3000);
```

This is a simple setup.
But image you got an Online Shop wich provides Data from your DB.
We can provide all our ShopItems to the Sitemap.xml, by filtering out the Data from
the Database and provide it to GoogleCrawler. So you don't need provide all Routes on your own.

## Crawl from the Database

````javascript
SHOPITEMS.find({}, '_id').then((SEO_TITLE) => {
  SEO_TITLE.forEach((item_id) => {
    sitemap.add({
      url: `/shop/items/${item_id}`,
      changefreq: 'daily',
      priority: 1,
    });
  });
});
````

### Providing Robots.txt 

Create a robots.txt file and porvide it as a static File in ../static/robots.txt.

1.) Add an express Route

````javascript 
server.get('/robots.txt', (req, res) => {
  res.sendFile(path.join(__dirname, '../static', 'robots.txt'));
});
````
path.join(__dirname) will directly point to the directory you are using

2.) content of this File

````javascript 
User-agent: *
Allow: /books/builder-book/
Disallow: /admin
````

User-agent: this tells the indexing Bots to crawl all Routes 
Allow: ...wich are behind a specific Route bsp /shop/items/
Disallow: ...this disallow a specfic Route for example /admin 
