---
title: "SQL Joining and Filtering"
date: 2021-09-02T16:49:44-04:00
draft: false
tags:
  - sql
---

I was working on a query this week where I wanted to join `Table A` to `Table B` on a unique primary key for a given date, keeping all rows from `Table A` (the left side) and enhancing any matches with the extra rows from `Table B` (the right side). Simple, right? Well, it is simple, but it's still easy to mess up if you're hurrying:

```
SELECT *
FROM tableA
LEFT JOIN tableB
ON tableA.org_id = tableB.org_id
WHERE tableA.dateA = 20210829
    AND tableB.dateB = 20210829;
```

This reads great in English when you scan through the statement, doesn't it? "Select everything from A joined to B on the unique key, keeping everything from A. And do it for August 29th, 2021."

The problem is that the query doesn't actually do what we just said. If you run this query, you will actually lose rows from `Table A` even though you used `LEFT JOIN`, which is supposed to be a magic ward that protects rows on the left-hand side from getting dropped. How can you lose results from the left side when you're using `LEFT JOIN`? Let's check it out:

Table A:
|org_id | valueA | dateA |
--- | --- | ---
|1|far|19001028|
|2|foo|20210829|
|3|bar|20210829|
|4|baz|20210829|

Table B:
|org_id | valueB | dateB |
--- | --- | ---
|2|flim|20210829|
|3|flam|20210829|

When you run the query above, it first left-joins up on the primary key:

|org_id | valueA | valueB | dateA | dateB |
--- | --- | --- | --- | ---
|1|far|NULL|19001028| NULL|
|2|foo|flim|20210829| 20210829|
|3|bar|flam|20210829| 20210829|
|4|baz|NULL|20210829| NULL|

And then it filters where both `dateA` and `dateB` are `20210829`:

|org_id | value | dateA | dateB |
--- | --- | --- | ---
|2|foo|20210829| 20210829
|3|bar|20210829| 20210829

So much for the magic protective powers of `LEFT JOIN`. We lost two of our rows from `Table A`, and one of them (with an `org_id` of `4`) had the date value we wanted to keep!

What do we want to do instead? Let's check this out:

```
SELECT *
FROM tableA
LEFT JOIN tableB
ON tableA.org_id = tableB.org_id
  AND tableA.dateA = tableB.dateB
WHERE tableA.dateA = 20210829;
```

Now what happens? In English is this equivalent to "Select everything from A joined to B on the unique key, when the dates are the same, and when the date is August 29th, 2021, keeping everything from A."

|org_id | valueA | valueB | dateA | dateB |
--- | --- | --- | --- | ---
|2|foo|flim|20210829| 20210829
|3|bar|flam|20210829| 20210829
|4|baz|NULL|20210829| NULL

Basically with the first query you're joining tables first and then filtering out the results second. So your `LEFT JOIN` isn't going to do anything to keep results that will get dropped by the filter.

In the second query, you're extending the `LEFT JOIN` conditions to include the date, so the protections for the left-hand table will stay in effect.

I wasted more time on this than I should have. Next time I won't, and hopefully you won't, either.