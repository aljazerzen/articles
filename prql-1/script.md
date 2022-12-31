Hello, today i wanna talk about PRQL, a sequel to SQL

Even though SQL is an old language that stood the test of time,
it leaves a lot be desired. 
This may not be noticable with short transactional queries,
but gets obvious when composing anything heavier

Let me show you an example.

Ill be using chinuk sample database, that contains data from a music store.
Our task is to find top 10 albums by revenue generated. For an album, we need to include all tracks and all of their sales.

For that, we will need these tables.

Ill first solve the task with SQL and then show you why I think PRQL does it better.

Lets first select all invoice lines.

Instead of quanity and price, we can compute revenue.

Before joining with other tables, lets first aggregate revenue by tracks. Note that along adding a group clause we need to jump back and modify select clause.

Now we can join with tracks, but because we have already used group, we need a new query and nest previous one as subquery.

Similarly as we did before, we can group and aggregate, but we have to make sure to remove track title, group clause will complain. 

To also get album titles, we have to nest and join again. Now it is only the matter of order and limit and we should have our result. 

To make query more readable, we can convert nested queries into common table expressions, but only if we are sure that our database engine can optimize it well.

Now lets do it again, this time with Pipelined Relational Query Language.
Syntax looks similar to SQL, but there is an important difference. From and select are independed functions that take their arguments and compose into a pipeline.
From specifies a table to pull initial relation from, and select takes a list of columns to select.

Derive is similar to select, but instead of replacing, it appends new columns at the back of the table. We have also assigned a new name to the computed column, which we could have also done in select. Because derive declares only one column, we can drop the list brackets, to be a little bit more concise.

Now let's use aggregate to compute total revenue. Again, we used a new name and again, we could drop the brackets. But this result is not what we want - instead of total revenue, we want a revenue of each track.

For that we need group function. This one is quite different than its SQL counterpart. It takes two arguments: first is a list of columns to group by and the second is a pipeline which is to be applied to each of the groups. 

The result of these two functions is a relation that has all the group columns followed by all the aggregate columns.

Join takes 2 arguments:
- table to join with ad
- join condition using comparison operator, similar to convential languages.
In our case, the condition compares columns with the same name, so we can use a special syntactic sugar: self equality operator.  
Join does an inner join by default, which is what we want, but just to be explcit we can specify side which is named argument. Named arguments dont care about order, so it could be placed after the condition or before the table, but i like it here. 

Now let's group, aggregate, and join again. Finally, we sort and limit to 10 results. Sort uses minus sign to sort rows descending, and unfortunatley, we do need brackets here, so minus does not get interepreted as a binary operation.

It's a good practice to alos add a select at the end of your query, because PRQL does not always have an explict list of columns that will be returned.

There are a few things I want to point out:
- this is all one pipeline. We could cut it after any function and it would still produce a valid relation. This makes it easy to explore, adapt and reuse your queries.
- PRQL removes a lot of that paper cuts that SQL has. For example, the flow of the query is always from top to bottom and when groupping you don't need to remove columns from previous select function. 

You may think that this adds a lot of unneeded table scans - for example where we select invoice id at the beginning of the pipeline and so we can inspect what happens under the hood.
PRQL compiler currenty targets SQL with possibility of adjusting the dialect. This means that it can be used with any SQL database and that we can inspect what it does with the familiar SQL.

Ok, so first of all we can that there are two CTEs, just as what we wrote before. Compiler will try to minimize the amount of CTEs produced, while making sure that the result is what you requsted. 

Then, there is no mention of invoice id. Because of clear language semantics, compiler can infer column lineage and determine that some columns are not needed to compute the final result.

Also take a look at joins that used double equals in PRQL and at order by clause that used minus instead of descending keyword.

Come and check out PRQL at our website, with an online compiler playground or a book with more examples.

Because we strive to integrate PRQL with many databases and tools, we built it as an opensource and free gift to all people dealing with data.

If you have ideas, questions or a will to help, join the discussion at our github repo.