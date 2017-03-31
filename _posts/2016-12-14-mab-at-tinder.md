---
layout: post
title:  "Multi-Armed Bandits at Tinder"
tags: machine_learning multi_armed_bandits tinder link
---

In a [post on Tinder's tech blog](http://tech.gotinder.com/smart-photos-2/), Mike Hall presents a new application for multi-armed bandits. At Tinder, they started to use multi-armed bandits to optimize the photo of users that is shown first: While a user can have multiple photos in his profile, only one of them is shown first when another user swipes through the deck of user profiles. By employing an adapted epsilon-greedy algorithm, Tinder optimizes this photo for the "Swipe-Right-Rate". Mike Hall about the project:

> It seems to fit our problem perfectly. Let’s discover which profile photo results in the most right swipes, without wasting views on the low performing ones. ...

> We were off to a solid start with just a little tweaking and tuning. Now, we are able to leverage Tinder’s massive volume of swipes in order to get very good results in a relatively small amount of time, and we are convinced that Smart Photos will give our users a significant upswing in the number of right swipes they are receiving with more complex and fine-tuned algorithms as we move forward.