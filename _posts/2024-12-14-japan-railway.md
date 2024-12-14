---
title: "Express Routes in Data and Travel: The Skip List Analogy"
date: 2024-12-14
layout: post
---

My wife and I spent last week in Japan. As the legends says, the railway and subway systems in Japan are incredibly complicated. We couldn't have navigated without Google Maps. There is a concept called "れっしゃしゅべつ" (train type). In short, trains can have multiple types. Some of them stop at every station (e.g. "普通 (local)"), while others may only stop at select stations (e.g. "急行 (express)"). Naturally, with fewer stops, the train is faster.

I didn't pay much attention to this when I first saw this in a metro station. However, when we took the Keihan Railway (京阪電車) from Osaka to Kyoto, I begin to notice that it was quite similar to [Skip List](https://en.wikipedia.org/wiki/Skip_list). (yeah, I was bored during the 45 minutes trip.)


## Skip List: A Quick Overview

>a skip list (or skiplist) is a probabilistic data structure that allows $O(\log n)$ average complexity for search as well as $O(\log n)$ average complexity for insertion within an ordered sequence of $n$ elements.

![Skip List](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2c/Skip_list_add_element-en.gif/400px-Skip_list_add_element-en.gif)

The basic idea is to create multiple levels of linked lists. The "base level" contains all the elements. For each element (except the head and tail), there's a certain probability (e.g. 50%) of "growing" to the next level. To perform a search, we start from the top level at the head, move as far forward as possible at that level, and go down to the next level if needed.

Let's use searching 80 for an example.

1. We start with the head at level 4. Since head (30) is less than 80, we move forward. However, since the next "stop" is nil, we go down to 30 at level 3.
2. At level 3, we move to 50 (the last element < 80). Since the next stop is nil, we go to 50 at level 2.
3. At level 2, we move 70 following the same logic, and then proceed to 70 at level 1.
4. At level 1, since the next stop is 90, we know we've reached the point where the search stops.

## Keihan Railway

![Keihan Railway](https://i.imgur.com/89a4NuB.png)

This is the route map of Keihan Railway. It also has several "levels". The "base level" stops at every station. The higher the level, the fewer stations it stops at. To see how it works, let's say we want to travel from Kyobashi to Owada. The optimal way is

- Take the Rapid Exp at Kyobashi station.
- Switch to Semi-exp at Moriguchishi station.
- Get off at Owada.

This pattern is quite similar to a skip list, isn't it?

Unlike a skip list, it's not practical to randomly pick stations for each level. Many factors need to be considered, such as the distance between stations, passenger demand, and operational efficiency. It would also be interesting to explore the model they use to design such systems.

## Summary

It's fascinating to observe the similarities between a data structure and a seemingly unrelated real-world system. These two were designed independently (according to ChatGPT), but we can always draw inspirations from other systems in our design.

> 世事洞明皆学问 / A thorough understanding of worldly affairs is knowledge.
