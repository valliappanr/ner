# Named Entity Recognition from News Feeds

## Overview

One of the challenges in online data is the trust worthiness. News feeds from well recognized websites(like BBC) can provide us high data quality.

List of things to process the news feeds and identify the named entity from the text are

- Consuming the news feed
- Next to parse the news feed and  extract the news feed links from the feed.
- Extract the text from the news
- Named entity could be extracted using different AI models, one of the model which provides more accurate result is using BERT based Named entity recognition.
- Store the extracted entity information to a database.
- Present the extracted information to the user.



## Architecture

There are many ways design the overall system. Using a single monolithic application. Or individual smaller microservices, each microservice can work independently to process its activity. The architecture below is using microservice architecture.

Individual microservice component is loosely coupled and all of them interact with each other using kafka topics.

![](C:\Users\valli\OneDrive\Desktop\news-feed.PNG)



### News Feed:

RSS stands for Really simple syndication, simple standardized content distribution to be up to date with the newscasts.

RSS feed producer produces the feed file in XML format and the sample rss file from BBC is below,



```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss xmlns:dc="http://purl.org/dc/elements/1.1/" version="2.0">
  <channel>
    <title>BBC News - Home</title>
    <link>https://www.bbc.co.uk/news/</link>
    <description>BBC News - Home</description>
    <language>en-gb</language>
    <copyright>Copyright: (C) British Broadcasting Corporation, see http://news.bbc.co.uk/2/hi/help/rss/4498287.stm for terms and conditions of reuse.</copyright>
    <pubDate>Thu, 26 Mar 2020 22:41:56 GMT</pubDate>
    <dc:date>2020-03-26T22:41:56Z</dc:date>
    <dc:language>en-gb</dc:language>
    <dc:rights>Copyright: (C) British Broadcasting Corporation, see http://news.bbc.co.uk/2/hi/help/rss/4498287.stm for terms and conditions of reuse.</dc:rights>
    <image>
      <title>BBC News - Home</title>
      <url>https://news.bbcimg.co.uk/nol/shared/img/bbc_news_120x60.gif</url>
      <link>https://www.bbc.co.uk/news/</link>
    </image>
    <item>
      <title>Coronavirus: UK government unveils aid for self-employed</title>
      <link>https://www.bbc.co.uk/news/uk-52053914</link>
      <description>The self-employed will be paid up to £2,500 a month to help them cope with the coronavirus crisis.</description>
      <pubDate>Thu, 26 Mar 2020 20:30:41 GMT</pubDate>
      <guid isPermaLink="false">https://www.bbc.co.uk/news/uk-52053914</guid>
      <dc:date>2020-03-26T20:30:41Z</dc:date>
    </item>
  </channel>
</rss>    

```

As each news item is within the <item> block, which contains when the news is published, title, description and the link to the detailed information about the news.



The news feed component polls the feed xml file and stores the file in an NFS share path. Sample java implementation is below,

```java
import com.sun.syndication.feed.synd.SyndEntry;
import com.sun.syndication.feed.synd.SyndFeed;
import com.sun.syndication.io.FeedException;
import com.sun.syndication.io.SyndFeedInput;
import com.sun.syndication.io.XmlReader;

import java.io.IOException;
import java.net.URL;


public class SyndicateReader {
    public SyndFeed getEntries(String url) throws IOException, FeedException {
        URL feedSource = new URL(url);
        SyndFeedInput input = new SyndFeedInput();
        SyndFeed feed = input.build(new XmlReader(feedSource));
        return feed;
    }

    public String getEntryId(SyndEntry entry) {
        String[] link = entry.getLink().split("/");
        int length = link.length;
        return link[length -1];
    }
}
```



SyndicateReader downloads the XML feed and also has the utility function to extract the links inside the feed file.



```java
import com.sun.syndication.feed.synd.SyndEntry;
import com.sun.syndication.feed.synd.SyndFeed;
import com.sun.syndication.io.FeedException;
import com.sun.syndication.io.SyndFeedOutput;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.FileWriter;
import java.io.Writer;

import java.util.List;
import java.io.IOException;

public class FeedConsumer {

    private String feedUrl;
    private String outputPath;
    private SyndicateReader syndicateReader;
    private static final Logger LOGGER = LoggerFactory.getLogger(FeedConsumer.class);
    public FeedConsumer(String feedUrl, String outputPath) {
        this.feedUrl = feedUrl;
        this.outputPath = outputPath;
        this.syndicateReader = new SyndicateReader();
    }
    public void consumeFeed(long feedTime) throws IOException, FeedException {
        SyndFeed syndFeed = syndicateReader.getEntries(feedUrl);
        List<SyndEntry> syndEntries = syndFeed.getEntries();
        writeFeedToFile(syndFeed, feedTime);
    }

    void writeFeedToFile(SyndFeed syndFeed, long timeInMillis) throws IOException, FeedException {
        SyndFeedOutput output = new SyndFeedOutput();
        String outputFilePath = String.format("%s/%s.xml", outputPath,
                timeInMillis);
        LOGGER.info("writing feed to {}", outputFilePath);
        Writer writer = new FileWriter(outputFilePath);
        output.output(syndFeed, writer);
    }

    public String getOutputPath() {
        return outputPath;
    }
}
```

FeedConsumer uses SyndicateReader to download the feed file and stores it in the shared file system. This component need to be highly available as this is the input process to the entire system.



******

[Next](Feed-processor.md)



