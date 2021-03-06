---
title: Batching queries for performance
author: jhutchings1
ms.author: justhu
ms.date: 04/12/2017
ms.topic: article
ms.prod: healthvault
description: Improve performance by batching many queries using a HealthRecordSearcher
---

Batching queries for performance
================================

When you first start using the HealthVault SDK, you might write code like this, based on what you have seen before:

```c#
HealthRecordSearcher searcher = PersonInfo.SelectedRecord.CreateSearcher();
HealthRecordFilter filter = new HealthRecordFilter(Height.TypeID);
searcher.Filters.Add(filter);
HealthRecordItemCollectionitems = searcher.GetMatchingItems()[0];
```

So, what's up with indexing into the result from GetMatchingItems()? Why isn't it simpler?

The answer is that queries can be batched up into a single filter, so that you can execute them all at once. You can write the following:

```c#
HealthRecordSearchersearcher = PersonInfo.SelectedRecord.CreateSearcher();
HealthRecordFilter filterHeight =new HealthRecordFilter(Height.TypeId);
searcher.Filters.Add(filterHeight);
HealthRecordFilterfilterWeight = new HealthRecordFilter(Weight.TypeId);
searcher.Filters.Add(filterWeight); 
ReadOnlyCollection<HealthRecordItemCollection> results = searcher.GetMatchingItems();
HealthRecordItemCollectionheightItems = results[0];
HealthRecordItemCollection weightItems = results[1];
```

Performance advantages of batching queries
------------------------------------------

The following test compares fetching 32 single Height values serially and batched together.

| Batch Size | Time in seconds |
|------------|-----------------|
| 1          | 0.98            |
| 2          | 0.51            |
| 4          | 0.28            |
| 8          | 0.16            |
| 16         | 0.10            |
| 32         | 0.08            |

If you need to fetch four different items, it's nearly four times faster to batch up the fetch compared to doing them independently. Why is this so big?

Doing a fetch involves the following steps:

1.  The request is created on the Web server.
2.  The request is transmitted across the Internet to HealthVault servers.
3.  The request is decoded, executed, and a response is created.
4.  The response is transmitted back to the Web server.
5.  The Web server unpackages the response.

When a filter returns small amounts of data, steps 1, 3, and 5 are fairly fast, but steps 2 and 4 involve network latency, which dominates the elapsed time. Batching eliminates those chunks of time, resulting in faster response time. As you fetch more data in each request, batching becomes less useful.

This table shows data for fetching 16 items:

| Batch Size | Time in seconds |
|------------|-----------------|
| 1          | 1.40            |
| 2          | 0.91            |
| 4          | 0.66            |
| 8          | 0.49            |
| 16         | 0.42            |
| 32         | 0.39            |
