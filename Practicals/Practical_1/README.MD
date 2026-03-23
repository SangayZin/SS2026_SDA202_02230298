# System Design: URL Shortener (TinyURL)

## Partical Overview

In this doc, i have broken down the TinyURL system design from the provided chapter. I looked into how the problem was framed, and where the proposed solution hits the mark and where it misses and what else to consider to make it better



## 1. Problem Statement Analysis

The goal fo this project is to develop a URL shortener like TinyURL. the system has two main requirements: accept a long URL and produce a short URL from the long URL; when someone clicks on the short URL link it redirect them to the orginal long URL.

To begin to develop this system, the author asks several questions about it and one important aspect is that the traffic volume is extremely high, and according to the author, every day there are approximately 1,160 writes per second. Using a read/write ratio of 10:1, the authors assume that they will receive approximately 11,600 reads per second

The system must hold an expected ten years worth (approximately 365 billion records). Given the average size of 100 bytes for each URL, the total data over the ten year period would be approximately 365 TB.

The characters that can be used in creating short URLs include the numbers (0-9), the lowercase letters (a-z) and the uppercase letters (A-Z). Therefore, there are 62 characters available to create a short URL. With 7 characters, 3.5 trillion links can be created and there are more than enough combinations to create (trillions) of unique links.

To keep things simple, it is assumed that a short URL once it is created can never be deleted or edited and additionally, the system must be highly available, scalable and fault-tolerant.

In summary, I believe the problem is well defined, and the data specified is adequate (the quantities provided create a clear measurement of success). My only small concern is the number of bytes that have been allotted (100) for each URL. Current real-world URLs are typically much longer as they include various tracking parameters, therefore the storage allocation will likely have to be adjusted higher.


## 2. Analysis of the Author's Proposed Solution

The author begins by designing a simple API with two types of requests: POST for short URLs and GET for redirection. They also discuss the trade-offs between using 301 redirects, which put more load on the server, and 302 redirects, which are better for analytics.

Initially, the author uses a hash table for storage, but soon realizes this won't scale and switches to a relational database with a simple table of items: an ID, short URL, and long URL.

The major challenge of the project is to create a 7-character short URL. The author explores two options:

1. **Hash + collision resolution**: the use of CRC32 or MD5 hashing and keeping only the first 7 characters of the generated output. The lack of an ID generator means things are stored in the hash table first, then redirected back to the long URL. However, performance issues arise in cases where there are hashing collisions.

2. **Base 62 conversions**: uniquely generating an integer for each long URL and then converting that integer to a 7-character short URL using Base 62. Because hashing is not used, there are no collision issues with this approach; however, sequential integers can lead to a security concern due to the ability to guess the next integer (short URL) in the series.

Ultimately, the author chooses to go with the Base 62 conversions and keep things simple, first checking if the long URL already exists; if not, creating a new ID, then converting the ID to the short URL, and finally storing it in the database. The author also incorporates caching to reduce the number of reads to the database during the redirection process.


## 3. Critical Review

### What I think works well

Two main areas where the author's approach works: separated read path (redirect) and write path (shorten). In the case of read vs. write, reads have different performance requirements than writes. There are more read requests than write requests thus, being faster is critical for reads however, reliability is paramount for the writes.

Using cache for redirecting traffic is a smart decision too. Since there are approximately 11,600 read requests per second, it would not work to go to the database each time. By caching the most frequently requested items in memory, the performance is dramatically improved and the load on the database is reduced.

Additionally, I like the comparison between 301 and 302 redirects, demonstrating they understand the trade-offs in the 'real world' versus creating solutions that work from a technical standpoint.

### Where I Have Some Doubts

Doubtful about analytics 302 vs 301

The Author thinks that the 302 redirect is better for analytics; however, with 100 million new urls created every day there would be an extremely large # of clicks. If all of those clicks are hitting my servers first then my infrastructure bill will be large. Perhaps, I could use a hybrid approach (i.e., use 302 for a few of the links only).

Database Scaling

The author believes that using a relational database is fine; however, with 365 billion records in the database I am not sure how this will work. If I shard the database by id's then I can look up short urls easily. However, when I shorten a long url I have to query the database to see if the long url already exists. This is not going to work because the data would be split out to several hundred servers and to get the long url I would need to query every shard, which is slow and expensive. The author does not address this gap.

Base62 Security Concern — Mentioned but not solved

The author has noted that using sequential id's will make it easy to guess the next short url in the sequence, which is a privacy issue since anyone can scrape every link possible. However, the author did not provide a solution like implementing a random number generator or using non-sequential ids.

I think the storage may be large

The other storage that would exist for this company, other than URL storage, would be databases; and the way in which databases can speed up search times is by storing the indexes, so the total storage could be 2-3 times the listed storage sizes. This isn’t a huge deal but worth thinking about.

## 4. What I Liked to Explore Further

The design of this project has given me an interest in several areas, which include:

#### Distributed ID Generation 
 I want to learn how distributed ID generators (like snowflake) can create unique ids across many machines and systems without needing a central point of failure. Specifically, I want to investigate how those systems utilize timestamps, machine ids, and sequence numbers to generate unique ids in a reliable way.

#### Database Sharding and Duplicate Detection 
 I found that this was one of the largest gaps in my knowledge of the design of this project and how database sharding will affect the duplicate check of long URLs. I would like to find additional solutions – such as bloom filters, or a separate lookup table – to efficiently check for duplicates when using database sharding.

#### Caching Strategy 
 Since there are billions of URLs that can be generated in this type of system, it will not be possible to cache all URLs. I would like to know how to determine which URLs remain in the caching system, possibly through the use of least-recently-used algorithms or focusing on the most popular URLs.

#### Rate Limiting 
Because anyone can create short URLs, this type of system is at risk of being abused by malicious users that can generate numerous URLs in a very short amount of time. I would like to learn more about how to effectively use token-bucket algorithms, or sliding window algorithms, to prevent an abuse of this type of system.

## 5. Summary

This high-level design ideas a massive-scale solution but has some issues . The database scaling is not clear enough, such as how to check for duplicate long URLs when we shard data. Using 302 redirects may increase the cost of infrastructure without providing an alternative . The security problem with the sequential Base 62 IDs is mentioned but not solved . Overall, while it is a good foundation , additional thought needs to be given to the database architecture, security and cost trade-offs prior to being ready for production.



