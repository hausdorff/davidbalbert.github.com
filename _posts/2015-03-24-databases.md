---
layout: post
title: "FoundationDB proves your primary datastore is the worst place in your stack to bet on new tech"
permalink: databases.html
comments: true
analysis: ■
---


FoundationDB [has been acquired by Apple](http://techcrunch.com/2015/03/24/apple-acquires-durable-database-company-foundationdb/). A notice [on their community site](http://community.foundationdb.com/) explains that they have pulled download links, and their client libraries now [return 404 on GitHub](https://github.com/FoundationDB/sql-layer-adapter-jdbc).

To database customers, this is a good lesson: assuming FDB did not coordinate with customers ahead of time, this instantly cost at least some FDB customers millions of dollars. It will be months of painful work to migrate, and now they must abandon the person-years of operational expertise they worked hard to accumulate.

In other words, **the worst possible place to take a risk in your production applications is your primary datastore,** and that is _especially_ true if it is client-facing. If you incur catastrophic data loss, or if the company goes under, it might cost you your entire business.

This is just a simple example of a trend: trusting a company to handle all of your mission-critical client data is a _huge_ commitment, and when you evaluate options, you need to be truly paranoid. You must ask hard questions, like _will I ever lose data?_ _Are the people trustworthy?_ _How do I know, and what guarantees do I have?_ Your fate is in their hands.

To database makers, this is a cautionary tale: do not be that company. When companies trust you with something as critical as this, it is your moral imperative to be a good citizen. You would do well to go beyond goodwill, and make formal ongoing support a commitment for a couple of years out, at least.

































