## Feed Processor

Feed processor consumes the feed document file link from the kafka topic.  Then extracts the link detail from the feed file, checks with redis cache whether the link had been previously processed, if not download the link and extract the sentences from the downloaded content. Some of the document can be huge and won't be able to process the entire document at once with neural networks for named entity recognition. 

![](C:\Users\valli\OneDrive\Desktop\feed-processor.PNG)

A sample feed item is below which contains the link about the details of the news feed.

```xml
    <item>
      <title>No British Grand Prix without quarantine exemptions</title>
      <link>https://www.bbc.co.uk/sport/formula1/52723200</link>
      <description>Formula 1 says it will be unable to hold a British Grand Prix if personnel are not given exemptions from plans to quarantine international travellers.</description>
      <pubDate>Tue, 19 May 2020 10:13:14 GMT</pubDate>
      <guid isPermaLink="false">https://www.bbc.co.uk/sport/formula1/52723200</guid>
      <dc:date>2020-05-19T10:13:14Z</dc:date>
    </item>
```



Most of the time, the contents of the link won't talk about just that news alone. There would be news related to the content what the news is about and also external links. The sentence extraction process need to take care of this.



The link (https://www.bbc.co.uk/sport/formula1/52723200) is processed and extracted the information related only to that document and the sentences within the document are grouped to certain length as below.



```
British Grand Prix: no F1 race without quarantine exemptions - BBC Sport British Grand Prix: no F1 race without quarantine exemptions By Andrew Benson Chief F1 writer 19 May From the section Formula 1 says it would be unable to hold a British Grand Prix if personnel are not given exemptions from plans to quarantine international travellers. The UK government will "soon" impose a requirement on all arrivals from abroad to self-isolate for 14 days. An F1 spokesman said: "A 14-day quarantine would make it impossible to have a "Additionally, it has a major impact on literally tens of thousands of jobs linked to F1 and supply chains."
A DCMS spokesman said no decisions on exemptions had yet been reached. F1 has drawn up plans to ensure its races Teams will be kept apart from each other at the tracks, and stay in separate hotels, to which they will be driven in buses. In addition, all personnel would be tested before travelling and every two days while at the races. The spokesman said: "We would be travelling back to the UK on F1-only occupied aircraft and all staff would be tested, making a quarantine totally unnecessary. "If all elite sport is to return to TV, then exemptions must be provided." The government's plans also threaten football's attempts to revive the Champions League this summer.
BBC Sport understands that F1's plans to start the season with two races in Austria on 5 and 12 would be able to go ahead, but the British races planned to follow on after that would have to be dropped. They would be replaced by other races in Europe, with the teams returning to the UK for two weeks before travelling again, until the F1's season is on ice as a result of the coronavirus crisis. Ten races have so far been postponed or cancelled, but F1 chairman Chase Carey has said he is "increasingly confident" of being able to stage a season of about 16 races starting in July.
Current plans involve the two races in Austria, then two events at either Silverstone - or Hockenheim in Germany if the UK proves impossible. After that, there would be a selection of four races from the previously scheduled grands prix in Spain, Hungary, Belgium, Italy, France and the Netherlands, before moving on around the rest of the world. The plan is to try to fit in either Canada or Singapore if possible, before races in Azerbaijan and Russia and then on to China and Japan.
There would be grands prix in the US, Mexico and Brazil if the coronavirus situation allows, then Vietnam, and then the season would end with consecutive races in Bahrain on 6 December and Abu Dhabi on 13 December. However, the number of variables involved means plans remain flexible, although F1 hopes to announce a European season at least in the next two to three weeks.
```



Have used Java library jsoup to parse the html document and extract sentences out of the link.

Sample code to remove the extra information from the downloaded html document from the link.

```
  String extractBody(String linkUrl) throws IOException {
        Document document = Jsoup.connect(linkUrl).get();
        document.select("a").remove();
        document.select("script").remove();
        document.select("div[class=orb-footer]").remove();
        document.select("section").remove();
        document.select("form").remove();
        document.select("figure").remove();
        document.select("img").remove();
        return document.text();
    }
```



[Next](Entity-extractor.md)