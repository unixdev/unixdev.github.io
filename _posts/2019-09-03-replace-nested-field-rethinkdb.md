---
layout: post
title: "RethinkDB Recipes"
tags: [development, javascript, rethinkdb]
---

[RethinkDB](https://rethinkdb.com) is an amazing database. It's a NoSQL document database with realtime push. It enables you to make some amazing applications with ease. It has a clean and elegant API. It's obvious the designers and authors behind it were geniuses. Too bad they couldn't make a viable business out of it. The project is in a stasis but it's still a solid piece of software that you can build upon.

This post is a random collection of ReQL snippets. These were useful to me in my projects. The official RethinkDB documentation does not list these directly. I found them after some tinkering.

## Tables
All the recipes in this post are taken from a particular project. The relevant tables and their subset of properties are:

### Table: missions
The main table in this project is `missions` with the following structure:
```json
{
	"id": "fb7d529a-eb21-4ff3-bfc1-efd8053a30e4",
	"status": "done",
	"created_at": Fri Jun 08 2018 03:50:25 GMT+00:00,
	"created_by": "f285a7e6-f26e-4b4c-b0ad-7c41eb8742de"
}
```

### Table: reviews
This table records user reviews done on missions.
```json
{
	"id": "001d49b6-de2c-431d-8e25-197ddb31dbd3",
	"mission_id": "fb7d529a-eb21-4ff3-bfc1-efd8053a30e4",
	"user_id": "e38a924d-7b42-4872-80a4-de1225c0fba2",
	"comment": "good work"
}
```

### Table: activities
This table is used to record user activities done on various documents.
```json
{
	"action_type": "mission_review",
	"table_name": "missions",
	"table_id": "fb7d529a-eb21-4ff3-bfc1-efd8053a30e4"
	"data": {
		"review": "good work"
		"review_id": "001d49b6-de2c-431d-8e25-197ddb31dbd3"
	}
}
```

## Data Explorer Oddity
The built-in Data Explorer of the RethinkDB admin interface uses JavaScript syntax. For some reason, you must always specify the database whenever referring to a table in a query. This is specific to Data Explorer, you don't have to do it in regular JavaScript code. For example, when doing a join, this will *not* work:
```javascript
r.db('my_db')
  .table('missions')
  .eqJoin('id', r.table('reviews'), {index: 'mission_id'})

```
Note: there is an index in the `missions` table named `mission_id` on the `mission_id` field.

You will have to specify the database as well like below:

```javascript
r.db('my_db')
  .table('missions')
  .eqJoin('id', r.db('my_db').table('reviews'), {index: 'mission_id'})

```

## Delete a Nested Property
Deleting a property is easy with the `replace` function. But how can you delete a nested property? The following does *not* work:
```javascript
r.db('my_db')
  .table('activities')
  .replace(function (act) {
	  return act.without('data.review_id')
  })
```

You need to write the query like this:
```javascript
r.db('my_db')
  .table('activities')
  .replace(function (act) {
	  return act.without({data: {review_id: true}})
  })
```

You have to say it's elegant! It's just not documented well.

## Update with an Inner Query
When making the `activities` table, we forgot to add the `review_id` property in it. We needed it later. We wanted to pluck the `id` of the `reviews` table and put them in the `activities` table too because the application will have multiple reviews on a mission.

We could do this with a script. But ReQL is quite powerful. We can do an inner query inside an update function! This is definitely not a fast operation, but this is the best we can do given the structure we have.

```javascript
r.db('my_db')
  .table('activities')
  .filter({action_type: 'mission_review'})
  .update({
      data: {
        review_id: r.db('my_db')
        	.table('reviews').getAll(r.row('table_id'), {index: 'mission_id'}).nth(0)('id')
      }
  }, {nonAtomic: true})
```

Notice that `getAll` returns a sequence. In our case, the sequence would always have a single element, so we could pluck the 0th element.

