# SQL practice using Sakila schema test database

### Appendix: List of Tables in the Sakila DB

* A schema is also available as `sakila_schema.svg`. Open it with a browser to view.

```sql
	'actor'
	'actor_info'
	'address'
	'category'
	'city'
	'country'
	'customer'
	'customer_list'
	'film'
	'film_actor'
	'film_category'
	'film_list'
	'film_text'
	'inventory'
	'language'
	'nicer_but_slower_film_list'
	'payment'
	'rental'
	'sales_by_film_category'
	'sales_by_store'
	'staff'
	'staff_list'
	'store'
```



* 1a. Display the first and last names of all actors from the table `actor`. 

```
	SELECT first_name, last_name
	from actor;
```

* 1b. Display the first and last name of each actor in a single column in upper case letters. Name the column `Actor Name`. 

```
	Select upper(concat(first_name,' ',last_name)) as 'Actor Name'
	from actor;
```

* 2a. You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." What is one query would you use to obtain this information?

```
	select actor_id, first_name, last_name
	from actor
	where first_name like 'Joe';
```

* 2b. Find all actors whose last name contain the letters `GEN`:

```
	select first_name, last_name
	from actor
	where last_name like '%GEN%';
```
  	
* 2c. Find all actors whose last names contain the letters `LI`. This time, order the rows by last name and first name, in that order:

```
	select first_name, last_name
	from actor
	where last_name like '%LI%'
	order by last_name, first_name;
```

* 2d. Using `IN`, display the `country_id` and `country` columns of the following countries: Afghanistan, Bangladesh, and China:

```
	select country_id, country
	from country
	where country in ('Afghanistan', 'Bangladesh', 'China');
```


* 3a. Add a `middle_name` column to the table `actor`. Position it between `first_name` and `last_name`. 

```
	alter table actor
	add column middle_name varchar(30) after first_name;

	select *
	from actor;

```
  	
* 3b. You realize that some of these actors have tremendously long last names. Change the data type of the `middle_name` column to `blobs`.

```
	alter table actor
	modify column middle_name blob;

	select *
	from actor;
```

* 3c. Now delete the `middle_name` column.

```
	alter table actor
	drop column middle_name;

	select * 
	from actor;
```

* 4a. List the last names of actors, as well as how many actors have that last name.

```
	select last_name as 'Last Name', count(last_name) as 'Last Name Count'
	from actor
	group by last_name;
```
  	
* 4b. List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors
 
```
	select last_name as 'Last Name', count(last_name) as 'Last Name Count'
	from actor
	group by last_name
	having count(last_name) > 1;
```

* 4c. Oh, no! The actor `HARPO WILLIAMS` was accidentally entered in the `actor` table as `GROUCHO WILLIAMS`, the name of Harpo's second cousin's husband's yoga teacher. Write a query to fix the record.

```
	select first_name, last_name
	from actor
	where first_name = 'Groucho' and last_name = 'Williams';

	update actor
	set first_name = 'HARPO'
	where first_name = 'Groucho' and last_name = 'Williams';

	select *
	from actor
	where last_name = 'Williams';
```

* 4d. Perhaps we were too hasty in changing `GROUCHO` to `HARPO`. It turns out that `GROUCHO` was the correct name after all! In a single query, if the first name of the actor is currently `HARPO`, change it to `GROUCHO`. Otherwise, change the first name to `MUCHO GROUCHO`, as that is exactly what the actor will be with the grievous error. BE CAREFUL NOT TO CHANGE THE FIRST NAME OF EVERY ACTOR TO `MUCHO GROUCHO`, HOWEVER!

```
	select first_name
	from actor
	where first_name = 'Harpo';

	update actor
	set first_name = 'GROUCHO'
	where first_name = 'Harpo';

	update actor
	set first_name = case
		when first_name = 'Harpo' THEN 'GROUCHO'
    	when first_name = 'Groucho' THEN 'MUCHO GROUCHO'
    	else first_name
	END;
	
	select *
	from actor;
```

* 5a. You cannot locate the schema of the `address` table. Which query would you use to re-create it? 

```
	create table address_new (
		address_id integer(11) NOT NULL,
    		address varchar(30) NOT NULL,
    		adress2 varchar(30) NOT NULL,
    		district varchar(30) NOT NULL,
    		city_id integer(11) NOT NULL,
    		postal_code integer(11) NOT NULL,
    		phone integer(10) NOT NULL,
    		location varchar(30) NOT NULL,
    		last_update datetime
	);
```

* 6a. Use `JOIN` to display the first and last names, as well as the address, of each staff member. Use the tables `staff` and `address`:

```
	select s.first_name as 'First Name', s.last_name as 'Last Name', a.address as 'Address'
	from staff as s
	join address as a 
	ON a.address_id = s.address_id;
```

* 6b. Use `JOIN` to display the total amount rung up by each staff member in August of 2005. Use tables `staff` and `payment`. 

```
	select concat(s.first_name,' ',s.last_name) as 'Staff Member', sum(p.amount) as 'Total Amount'
	from payment as p
	join staff as s
	on p.staff_id = s.staff_id
	where payment_date like '2005-08%'
	group by p.staff_id;
```
  	
* 6c. List each film and the number of actors who are listed for that film. Use tables `film_actor` and `film`. Use inner join.

```
	select f.title as 'Film', count(fa.actor_id) as 'Number of Actors'
	from film as f
	join film_actor as fa
	on f.film_id = fa.film_id
	group by f.title;
```
  	
* 6d. How many copies of the film `Hunchback Impossible` exist in the inventory system?

```
	select f.title as Film, count(i.inventory_id) as 'Inventory Count'
	from film as f
	join inventory as i
	on f.film_id = i.film_id
	where f.title = 'Hunchback Impossible'
	group by f.film_id;
```

* 6e. Using the tables `payment` and `customer` and the `JOIN` command, list the total paid by each customer. List the customers alphabetically by last name:

```
	select concat(c.first_name,' ',c.last_name) as 'Customer Name', sum(p.amount) as 'Total Paid'
	from payment as p
	join customer as c
	on p.customer_id = c.customer_id
	group by p.customer_id;
```

* 7a. The music of Queen and Kris Kristofferson have seen an unlikely resurgence. As an unintended consequence, films starting with the letters `K` and `Q` have also soared in popularity. Use subqueries to display the titles of movies starting with the letters `K` and `Q` whose language is English. 

```
	select f.title
	from film as f
	where f.language_id = (select language_id from language where name = 'English')
	and f.title like 'K%' or 'Q%' ;
```

or

```
	select f.title
	from film as f
	join language as l
	on f.language_id = l.language_id
	where f.title like 'K%' or 'Q%' and l.name = 'English';
```

* 7b. Use subqueries to display all actors who appear in the film `Alone Trip`.

```
	select CONCAT(first_name,' ',last_name) as 'Actors in Alone Trip'
	from actor
	where actor_id in 
	(select actor_id from film_actor where film_id = 
	(select film_id from film where title = 'Alone Trip'));
```
   
* 7c. You want to run an email marketing campaign in Canada, for which you will need the names and email addresses of all Canadian customers. Use joins to retrieve this information.

```
	select concat(c.first_name,' ',c.last_name) as 'Name', c.email as 'E-mail'
	from customer as c
	join address as a on c.address_id = a.address_id
	join city as cy on a.city_id = cy.city_id
	join country as ct on ct.country_id = cy.country_id
	where ct.country = 'Canada';
```

* 7d. Sales have been lagging among young families, and you wish to target all family movies for a promotion. Identify all movies categorized as famiy films.

```
	select f.title as 'Movie Title'
	from film as f
	join film_category as fc on fc.film_id = f.film_id
	join category as c on c.category_id = fc.category_id
	where c.name = 'Family';
```

* 7e. Display the most frequently rented movies in descending order.

```
	select f.title as 'Movie', count(r.rental_date) as 'Times Rented'
	from film as f
	join inventory as i on i.film_id = f.film_id
	join rental as r on r.inventory_id = i.inventory_id
	group by f.title
	order by count(r.rental_date) desc;
```

* 7f. Write a query to display how much business, in dollars, each store brought in.

```
	select store as 'Store', total_sales as 'Total Sales' from sales_by_store;

	select concat(c.city,', ',cy.country) as `Store`, s.store_id as 'Store ID', sum(p.amount) as `Total Sales` 
	from payment as p
	join rental as r on r.rental_id = p.rental_id
	join inventory as i on i.inventory_id = r.inventory_id
	join store as s on s.store_id = i.store_id
	join address as a on a.address_id = s.address_id
	join city as c on c.city_id = a.city_id
	join country as cy on cy.country_id = c.country_id
	group by s.store_id;

```

* 7g. Write a query to display for each store its store ID, city, and country.

```
	select s.store_id as 'Store ID', c.city as 'City', cy.country as 'Country'
	from store as s
	join address as a on a.address_id = s.address_id
	join city as c on c.city_id = a.city_id
	join country as cy on cy.country_id = c.country_id
	order by s.store_id;
```
  	
* 7h. List the top five genres in gross revenue in descending order. (**Hint**: you may need to use the following tables: category, film_category, inventory, payment, and rental.)

```
	select c.name as 'Film', sum(p.amount) as 'Gross Revenue'
	from category as c
	join film_category as fc on fc.category_id = c.category_id
	join inventory as i on i.film_id = fc.film_id
	join rental as r on r.inventory_id = i.inventory_id
	join payment as p on p.rental_id = r.rental_id
	group by c.name
	order by sum(p.amount) desc
	limit 5;
```
  	
* 8a. In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross revenue. Use the solution from the problem above to create a view. If you haven't solved 7h, you can substitute another query to create a view.

```
	create view top_5_genre_revenue as 
	SELECT c.name as 'Film', sum(p.amount) as 'Gross Revenue'
	from category as c
	join film_category as fc on fc.category_id = c.category_id
	join inventory as i on i.film_id = fc.film_id
	join rental as r on r.inventory_id = i.inventory_id
	join payment as p on p.rental_id = r.rental_id
	group by c.name
	order by sum(p.amount) desc
	limit 5;
```
  	
* 8b. How would you display the view that you created in 8a?

```
	SELECT *
	FROM top_5_genre_revenue;
```

* 8c. You find that you no longer need the view `top_five_genres`. Write a query to delete it.

```
	drop view top_5_genre_revenue;
```
