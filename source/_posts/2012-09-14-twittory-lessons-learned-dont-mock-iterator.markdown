---
layout: post
title: "Twittory lessons learned - don't mock Iterator"
date: 2012-09-14 11:40
comments: true
categories: 
---
~~So I'm doing this pet project now called [Twittory](https://github.com/pbuda/twittory) and because it's hosted on Github, people can review it and voice their oppinions.~~__Update (03 Feb 2013): I no longer develop this project.__

Marcin Deryło ([twitter](https://twitter.com/marcinderylo), [blog](http://marcinderylo.blogspot.com/)), a friend of mine, has recently reviewed some simple tests already present in the project and he raised an issue with mocking iterators.

I've had a very simple method which iterated over a collection returned by Twitter4J API. To do that, I simply mocked an Iterator and it's hasNext() and next() methods (using Mockito it's really simple anyway):
{% codeblock lang:java %}
@Test
public void returns_details_only_for_tweets_with_links() throws Exception {
    Status status1 = prepareStatus("Status 1");
    Status status2 = prepareStatus("Text with link http://www.onet.pl in it");
    Status status3 = prepareStatus("Status 3");
    ResponseList<Status> mockResponse = mock(ResponseList.class);
    Iterator mockIterator = mock(Iterator.class);

    when(mockIterator.hasNext()).thenReturn(true).thenReturn(true).thenReturn(true).thenReturn(false);
    when(mockIterator.next()).thenReturn(status1).thenReturn(status2).thenReturn(status3);
    when(mockResponse.iterator()).thenReturn(mockIterator);
    when(twitter.getHomeTimeline()).thenReturn(mockResponse);

    List<LinkDetails> scan = scanner.scan();
    assertNotNull(scan);
    assertEquals(1, scan.size());
}
{% endcodeblock %}
But said friend of mine suggested to drop mocking here in favor of actually returning a real iterator:
{%codeblock lang:java%}
@Test
public void returns_details_only_for_tweets_with_links() throws Exception {
    Status status1 = prepareStatus("Status 1");
    Status status2 = prepareStatus("Text with link http://www.onet.pl in it");
    Status status3 = prepareStatus("Status 3");
    ResponseList<Status> mockResponse = mock(ResponseList.class);

    when(mockResponse.iterator()).thenReturn(Arrays.asList(status1, status2, status3).iterator());
    when(twitter.getHomeTimeline()).thenReturn(mockResponse);

    List<LinkDetails> scan = scanner.scan();
    assertNotNull(scan);
    assertEquals(1, scan.size());
}
{% endcodeblock %}
Pretty slick, eh? Saves code and complexity while becoming much more readable at the same time.

Learning FTW.