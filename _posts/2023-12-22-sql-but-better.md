---
layout: post
title: SQL... but better
lead: I can't believe it's not SQL
categories: work
tags: Data, SQL, Malloy, PRQL, DuckDB
---

So, with my first actual technical blog - here's something that is really interesting, but probably very difficult to convince a team to take on board; but hey-ho it's interesting to me.

SQL, at least the DQL part is a declarative language created with the idea to sound english - this allows the basic structure of SQL

```sql
SELECT field1, field2
FROM table1
WHERE field1 > 100;
```

... to be read like (with some grammar stuff added) ...

```
Select these fields from this table where this field 
is greater than 100
```

... pretty much like for like.

which you know, is great and all, but this does [mean a lot of shortcomings and leaves many areas for improvement](https://www.scattered-thoughts.net/writing/against-sql) - and does lead to the statement

![There's got to be a better way!](https://c.tenor.com/YygntU2fx3kAAAAC/tenor.gif)

And there are in the corners of the internet, the following are a couple of ones that I've played around with, or have heard good things about - out of pure simplicity sake, I'm going to split the run through into 2 categories - friendly SQL & SQL alternatives.

# Friendy SQL

This category 