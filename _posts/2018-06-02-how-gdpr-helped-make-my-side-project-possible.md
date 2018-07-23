---
layout: post
current: post
cover: assets/images/first-data-viz/privacy.jpg
navigation: True
title: "How GDPR Helped Make My Side Project Possible"
date: 2018-06-02 23:00:00
tags: [tech]
class: post-template
subclass: 'post'
author: eric
permalink: /:categories/:year/:month/:day/:title/
---

In [my last post](http://ericbai.co/2018/05/10/i-made-some-data-visualizations-for-my-girlfriend/), I shared my first data viz project, [ChatStats](https://github.com/baieric/chatstats). Here I'll share a behind-the-scenes story about how the EU's General Data Protection Regulation (GDPR) allowed me to create ChatStats. Wild, I know.

It all starts with me trying to get a copy of my Facebook Messenger data.

### Attempting To Get My Chat Data

The first step in any data viz project is getting the data. In this case, I needed my message history in a format that a program could easily manipulate.

At first, I had a JS script running to scroll up through my chat every few milliseconds. This method of collecting my chat data would have taken days or weeks... per chat! Yikes.

In the final product, ChatStats uses [Facebook's Download Your Information (DYI) feature](https://www.facebook.com/dyi), which allows you to export your entire chat history in a JSON format. Funnily enough, Facebook revamped DYI in late April 2018, and I was trying to use it right before that. There were lots of problems with the old version:

- There was no JSON format available, only HTML
- The web interface to get your data was much uglier and provided less options
- There were bugs that caused some chat histories to be completely missing

Before the update, I filed a bug about DYI to my friend who was interning at Facebook at the time... so maybe I had something to do with it! Later, I read up on GDPR's new rules and, uh, nevermind, it was definitely GDPR that led Facebook to make these changes.

### How GDPR Played Into This

GDPR is a new European Union privacy law that went into effect on May 25, 2018. If you haven't heard of it, you definitely know about all the Privacy Policy update emails you got around that time, which were sent to comply with the new laws. Since large tech companies like Facebook operate globally, it makes sense for them to follow these new laws consistently around the world, rather than treat European users differently, for simplicity's sake.

Two particularly relevant rights that GDPR bestows on users are the right to access and the right for data portability.

**The right to access** stipulates that users have the right to access a copy of all the data a company has about them, even if they choose to deactivate their account. This provides the user transparency about what records the service provider is keeping. Facebook's old DYI tool already allowed users access to their data before GDPR, but it was very buggy and unpolished. In fact, the bugs that caused missing chat histories would have meant that the DYI tool did not properly comply with GDPR.

**Data portability** means that users must be able to receive their personal data in a machine-readable format. This gives users the ability to easily switch service providers. They can export their data off of the company’s servers and import it into a different service, preventing them from being locked in to a single service provider. To comply with GDPR, Facebook's DYI tool now supports a JSON format. This made the creation of ChatStats much more feasible, since the data was easier to use and required much less formatting and processing on my part.

These new rules prompted Facebook to dramatically improve its old, difficult-to-use DYI tool. And thanks to these changes, I was able to create ChatStats and make a cool blog post for my girlfriend!

### Conclusions

It turns out that when governments and activists fight for better data privacy, we also get to use our data in more powerful ways. Without Facebook's improved DYI feature, ChatStats couldn't have existed. I imagine Reddit's [/r/DataIsBeautiful subreddit](https://www.reddit.com/r/dataisbeautiful/) is going to get a lot of great new content thanks to GDPR.

As every major tech company is GDPR compliant by now, it's now easy to download your chat history in machine-readable formats from other apps like Instagram, WhatsApp, and Twitter. It would only take some minor tweaking to convert that data into the JSON format that ChatStats uses. (If you're interested in using ChatStats for chats in other applications, let me know!)

Neat projects like ChatStats are a fun by-product of the important work that governments and activists do to ensure the ethical use of our data! The relatively unregulated internet that we know now will look very different in the coming years. Hopefully, the internet will continue to change to further empower users.
