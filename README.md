# scrape-news

Scrape South African news to provide search with links to the original articles.

This is important because search engines make no guarantee of including all articles. It's important to be able to find news articles even if search engines didn't think they're worth indexing.

## Contribution

We need very broad coverage of news outlets. Contributed spiders are very welcome. Any South African news is welcome.

It's really easy to contribute spiders. Basically you can copy an existing spider and change the xpaths to find the elements we're extracting.

See the In Progress column at [https://trello.com/b/9TVRB4gb/public-people](https://trello.com/b/9TVRB4gb/public-people) to see which publications are currently being tackled to avoid duplication. 

Ideally spiders should be driven from the outlet's sitemap. Ideally you'll find the sitemap from /robots.txt. If you don't find it there, try /sitemap.xml or /sitemap.txt. If you can't find a sitemap, use a crawling spider (you can copy from `thenewage`).

Send a pull request or get in touch.

Next, go get started at [Development](#development)

## Copyright of news content

We do not make news content available for public consumption. We simply store and index the news content and the original URL and publication date to provide search functionality similar to search engines. This project intends to provide better access to news on the publisher's website. It should be used to send readers to relevant news websites rather than to replace them.

## Development

### Set up your development environment

Fork this repository on GitHub and clone your fork:
```bash
git clone https://github.com/your-name/scrape-news.git
```
or
```bash
git clone git@github.com:your-name/scrape-news.git
```
(Make sure to replace ```your-name```.)

Create a Python 2 virtual environment for this project inside the cloned project directory (Note: your virtual environment program might not be called `pyvenv` -
```bash
cd scrape-news
pyvenv env
```

If your default Python is Python3, try `virtualenv` instead; it creates a Python2 environment by default:
```bash
cd scrape-news
virtualenv .env2
source .env2/bin/activate
pip install -r requirements.txt
```
(When you're done later, use `deactivate` to leave your virtual environment.)

Run a scraper to check that your environment is working properly. The argument `since_lastmod` is the earliest sitemap file and page the scraper will include. The setting `ITEM_PIPELINES` disables the pipeline we have configured which you don't need for just developing a spider.

```bash
scrapy crawl iol -s ITEM_PIPELINES="{}" -a since_lastmod=2018-04-30
```

If it's working correctly, it will output a lot of information:

e.g. after starting up it will find the sitemap and some articles that it will ignore in the sitemaps:

```
2018-05-03 18:21:17 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.iol.co.za/robots.txt> (referer: None) ['cached']
2018-05-03 18:21:17 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.iol.co.za/sitemap.xml> (referer: https://www.iol.co.za/robots.txt) ['cached']
2018-05-03 18:21:18 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.iol.co.za/personal-finance/sitemap.xml> (referer: https://www.iol.co.za/sitemap.xml)
2018-05-03 18:21:18 [scrapenews.spiders.sitemap] DEBUG: Skipping too old https://www.iol.co.za/personal-finance/utmost-good-faith-is-the-cornerstone-of-insurance-14626128
2018-05-03 18:21:18 [scrapenews.spiders.sitemap] DEBUG: Skipping too old https://www.iol.co.za/personal-finance/facebooks-lesson-on-being-priced-for-perfection-14625488
2018-05-03 18:21:18 [scrapenews.spiders.sitemap] DEBUG: Skipping too old https://www.iol.co.za/personal-finance/six-cappuccinos-or-a-year-off-your-home-loan-14626132
```

when it reaches articles that are after the earliest accepted date, it will actually scrape content from the pages and print the resulting [ScrapenewsItem]() for the article

```
2018-05-03 18:21:34 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.iol.co.za/personal-finance/stanlib-may-further-reduce-fund-offering-14717297>
{'body_html': u'<div itemprop="articleBody" class="article-body" ... the ratings, De Klerk says.</p>\n<!-- C-ON- TEXT_CONTENT_END --></div></div>',
 'byline': u'Mark Bechard',
 'file_name': 'stanlib-may-further-reduce-fund-offering-14717297',
 'publication_name': 'IOL News',
 'published_at': '2018-04-30T15:04:00+02:00',
 'retrieved_at': '2018-05-03T16:21:31.603070',
 'spider_name': 'iol',
 'title': u'Stanlib may further reduce fund offering',
 'url': 'https://www.iol.co.za/personal-finance/stanlib-may-further-reduce-fund-offering-14717297'}
```

### Make a spider

First, add this repository as a remote and make sure your local master is up to date:
```bash
git remote add upstream git@github.com:public-people/scrape-news.git
git fetch upstream
git checkout master
git merge upstream/master
```

Create and check out a branch for the spider you're making:
```bash
git checkout -b newssite
```
(Replace ```newssite``` with the name of the publication you're making a spider for.)

Copy a spider from the repository and amend it as necessary. When it's done, ```git add``` it and ```commit``` the change to your working branch.

### Test your responses

To test individual xpath or css responses you can use the scrapy shell:
```bash
scrapy shell "https://www.newssite.co.za/article-url"
```
If you go to the same url in your browser and right-click on, say, the title of the article, and select 'Inspect Element (Q)', you'll see something like
```html
<h1 class="article-title">Title of article</h1>
```
highlighted. In the scrapy shell you can then enter
```bash
>>> response.css('h1.article-title').xpath('text()').extract_first()
```
to get the title. 

Now that you've checked that this works and doesn't have some unintended consequence, you can copy it into the relevant part of your spider; in this case:
```python
def parse(self, response):
    ...
    title = response.css('h1.article-title').xpath('text()').extract_first()
    ...
```

#### Using css instead of xpath for classes

You could have also used the xpath in the above:
```bash
>>> response.xpath('//h1/[@class="article-title"]/text()').extract_first()
```
but the preference is to use css lookup for classes.

The reason for this is that xpaths can be brittle in more complicated instances. Consider the case of
```html
<div class="byline pin-right">John Smith</div>
```
The only way an xpath query will work here is if you give the exact class name:
```bash
>>> response.xpath('//div/[@class="byline pin-right"]/text()').extract_first()
```
which means that if the same publication uses, say, ```byline pin-left``` for certain articles the spider won't get a response for 'byline'. 
(And if you were using ```extract()``` instead of ```extract_first()``` it would get an error.) 
The safer option is therefore to use the following instead:
```bash
>>> response.css('div.byline').xpath('text()').extract_first()
```

#### Make a pull request

Once your spider is ready for review, first make sure your local master is up to date:
```bash
git fetch upstream
git checkout master
git merge upstream/master
```
and merge any changes into your working branch:
```bash
git merge master newssite
```

Then push your working branch to your fork on GitHub (use ```git remote -v``` to check the names of your remotes):
```bash
git push origin newssite
```
Go to [Pull requests](https://github.com/public-people/scrape-news/compare), choose to 'compare across forks', and compare the ```base fork: public-people/scrape-news```, ```base: master``` to ```head fork: your-name/scrape-news```, ```compare: newssite```, and make a new pull request!   

## Deployment

We use scrapyd to run the scrapers.

We use cron to schedule the scrapers regularly.

### Deploy a scraper to scrapyd

### Schedule a scraper

SitemapSpider scrapers can run daily, fetching only the latest articles. Crawling scrapers have to visit every page on the site so we only run them weekly.

Tunnel a connection to the server if you're not scheduling it from the server:

```
ssh -L 6800:localhost:6800 username@hostname
```

#### Schedule a SitemapSpider

SitemapSpiders take an argument `since_lastmod` which is an ISO format date filtering sitemaps and links in sitemaps. To do a complete scrape, just set it to a date very long ago, like `1900-01-01`.

```
curl -v http://localhost:6800/schedule.json -d project=scrapenews -d spider=iol -d setting=ALEPH_API_KEY=... -d since_lastmod=$(date +%Y-%m-%d -d "5 day ago")
```

#### Schedule a crawling spider

```
curl -v http://localhost:6800/schedule.json -d project=scrapenews -d spider=thenewage -d setting=ALEPH_API_KEY=...
```