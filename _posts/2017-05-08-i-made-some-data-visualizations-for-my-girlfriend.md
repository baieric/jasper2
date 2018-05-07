---
layout: post
current: post
cover: False
navigation: True
title: 'I Made Some Data Visualizations For My Girlfriend'
date: 2017-05-08 11:00:00
tags: [tech]
class: post-template
subclass: 'post'
author: eric
---

I just finished my undergrad last month, and it's really sinking in how special and fleeting the past five years have been. I'm privileged to have had so many unique opportunities and experiences throughout these fourteen trimesters, both in my career and personal life. I've also met so many amazing people that I'm lucky to call my friends. In fact, I happened to meet my soon-to-be best friend during the first week of university, and now we've been dating for over four years.

I wanted to commemorate my relationship with Camille thus far, so I made some data visualizations of our Facebook messages. This involved analyzing over 152,000 messages that span from October 2013 up until April 2018. Let's dive in!

### Graphs

<img src="/assets/images/chatstats/sender_messages.png" alt="messages sent" style="max-height: 500px;"/>
<img src="/assets/images/chatstats/words_per_message.png" alt="words per message" style="max-height: 500px;"/>

Camille does tend to message more, but at least I have more words per message to slightly makes up for the difference. 😅

<img src="/assets/images/chatstats/weekday_messages.png" alt="messages by weekday" style="max-height: 500px;"/>
<img src="/assets/images/chatstats/time_in_day_messages.png" alt="messages by hour of day" style="max-height: 500px;"/>

Camille consistently messages me more at every hour and every day of the week... except for 5AM. This could be due to my bad sleep habits, or due to a timezone difference from when I spent 4 months in Singapore.

We also message noticeably less on Saturdays, probably because we're hanging out in person.

<img src="/assets/images/chatstats/per_term_messages.png" alt="messages by hour of day" style="max-height: 500px;"/>

As anyone from UWaterloo knows, our lives are divided into four months, where we alternate between school and co-op. I thought it would be pretty interesting to see how different our chat habits were every trimester.

This graph is super interesting. That first spike in Spring 2014 was during my first co-op in Toronto, so Camille and I were no longer living across the hall from each other like we did in first year. A large, sustained rise in messages began in Spring 2016. I'm not exactly sure why, but we did used to mainly text each other on SMS early on, and switched to only using Facebook at some point.

The trimesters with the most messages all occurred when we were super long-distance. I was in California in Fall 2016 and Fall 2017, and I was in Singapore in Winter 2017.

<img src="/assets/images/chatstats/emoji_total.png" alt="most frequent emoji" style="max-height: 500px;"/>

That's a lot of sad faces. I guess we're both stressed out and anxious a lot in university, so please don't read too much into it LMAO.

I'd also like to note that I'm clearly the more positive and affectionate person.

<img src="/assets/images/chatstats/top_stickers.png" alt="most frequent stickers" style="max-height: 500px;"/>

I'm constantly serenading Camille with romantic stickers, but she never reciprocates. Wish my gf's sticker game was stronger.

Facebook Foxes, Pusheen, and Sugar Cubs for life.

<img src="/assets/images/chatstats/names.png" alt="names said in chat" style="max-height: 500px;"/>

Why does Camille say my name so much? I know you're talking to me, we're the only two people here.

<img src="/assets/images/chatstats/sender_distinguishing_words.png" alt="most distinguishing words" style="max-height: 500px;"/>

To quantify distinctiveness of our words, I computed the [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) values of every word.

This was a fun graph that highlights how we talk differently. She likes to leave off the G in "-ing" words. I say "whoa" and she says "woah". I say "8:30" and she says "830". I didn't realize we complained about early classes so often for it to be statistically relevant though.

I also made a similar graph for most distinguishing words per trimester. It was a blast looking through it with Camille, but it actually turned out to be too personal to share here, haha.

These were all of the most interesting graphs I managed to make! Camille, thank you for all the experiences we've shared during university, and the many more to come. ❤️

### How To Make Your Own Graphs

If you want to make these graphs for your own Facebook messages, you're in luck. [**Check out ChatStats on Github to make your own data visualizations**](https://github.com/baieric/chatstats).

ChatStats even works with group chats, which are also incredibly fun to analyze.

The code is also easy to modify and extend, so you can easily put together new types of graphs.

If you do try out ChatStats, let me know on [Twitter](https://twitter.com/BaiEric) or in the comments!
