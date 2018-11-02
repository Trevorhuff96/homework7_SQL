--------------------------------------------------------------------------------------------------------
-- 										TREVOR HUFFSTETLER
----------------------------------------------------------------------------------------------------


# Unit 10 Assignment - SQL

### Create these queries to develop greater fluency in SQL, an important database language.

------------------------------------------------------------------------------------------------------------------------------
--                                SECTION 1                                        --
------------------------------------------------------------------------------------------------------------------------------



-- 1a. Display the first and last names of all actors from the table `actor` --

select first_name, last_name 
from actor;

-- 1b. Display the first and last name of each actor in a single column in upper case letters. 
-- Name the column `Actor Name`
select upper(concat( first_name," ", last_name)) as 'Actor Name'
from actor;



------------------------------------------------------------------------------------------------------------------------------
--                                SECTION 2                                       --
------------------------------------------------------------------------------------------------------------------------------


-- 2a. You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." 
-- What is one query would you use to obtain this information?
select actor_id as id, first_name, last_name
from actor
where first_name='JOE';
 
-- 2b. Find all actors whose last name contain the letters `GEN`
select *
from actor
where last_name like '%GEN%';

-- * 2c. Find all actors whose last names contain the letters `LI`. This time, order the rows by last name and first name, in that order:
select * 
from actor
where last_name like '%LI%'
order by last_name, first_name;

-- * 2d. Using `IN`, display the `country_id` and `country` columns of the following countries: Afghanistan, Bangladesh, and China:
select country_id, country
from country
where country IN ("Afghanistan","Bangladesh","China");


------------------------------------------------------------------------------------------------------------------------------
--                                SECTION 3                                        --
------------------------------------------------------------------------------------------------------------------------------



-- * 3a. You want to keep a description of each actor. You dont think you will be performing queries on a description, 
-- so create a column in the table `actor` named `description` and use the data type `BLOB` 
-- (Make sure to research the type `BLOB`, as the difference between it and `VARCHAR` are significant).

ALTER TABLE actor
ADD description BLOB(300);

--	3b. Very quickly you realize that entering descriptions for each actor is too much effort. Delete the `description` column.
ALTER TABLE actor
drop description;



------------------------------------------------------------------------------------------------------------------------------
--                                SECTION 4                                        --
------------------------------------------------------------------------------------------------------------------------------



--	4a. List the last names of actors, as well as how many actors have that last name.
select last_name, COUNT(last_name) as frequency
from actor
group by last_name;

--	4b. List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors.
select last_name, COUNT(last_name) as count
from actor
group by last_name
having count>=2;

--	4c. The actor `HARPO WILLIAMS` was accidentally entered in the `actor` table as `GROUCHO WILLIAMS`. Write a query to fix the record.
UPDATE actor
SET first_name='HARPO'
WHERE first_name='GROUCHO' and last_name ='WILLIAMS';

--- 4d. Perhaps we were too hasty in changing `GROUCHO` to `HARPO`. It turns out that `GROUCHO` was the correct name after all! 
--	In a single query, if the first name of the actor is currently `HARPO`, change it to `GROUCHO`.
--  make sure safe mode is off <- TJ
UPDATE actor
SET first_name =
CASE WHEN first_name='HARPO' THEN 'GROUCHO' end;



------------------------------------------------------------------------------------------------------------------------------
--                                SECTION 5                                        --
------------------------------------------------------------------------------------------------------------------------------
--	5a. You cannot locate the schema of the `address` table. Which query would you use to re-create it?

show create table address;






------------------------------------------------------------------------------------------------------------------------------
--                                SECTION 6                                      --
------------------------------------------------------------------------------------------------------------------------------


-- * 6a. Use `JOIN` to display the first and last names, as well as the address, of each staff member. 
-- Use the tables `staff` and `address`:

select staff.first_name, staff.last_name, address.address
from address
join staff on staff.address_id=address.address_id;

--	6b. Use `JOIN` to display the total amount rung up by each staff member in August of 2005. Use tables `staff` and `payment`.

select staff.first_name, staff.last_name, sum(payment.amount)
from payment 
join staff on staff.staff_id = payment.staff_id
where payment_date like '2005-08%'
group by staff.staff_id;


--	6c. List each film and the number of actors who are listed for that film. Use tables `film_actor` and `film`. Use inner join.

select film.title, count(actor_id) as num_actors
from film 
inner join film_actor on film_actor.film_id=film.film_id;

--	6d. How many copies of the film `Hunchback Impossible` exist in the inventory system?

select count(inventory.film_id) as Hunchback_Impossible_Count
from inventory
join film on film.film_id=inventory.film_id
where film.title= 'Hunchback Impossible';

--	6e. Using the tables `payment` and `customer` and the `JOIN` command, list the total paid by each customer. 
--	List the customers alphabetically by last name:

select customer.first_name, customer.last_name, sum(payment.amount) as total_amount_paid
from payment
join customer on customer.customer_id= payment.customer_id
group by customer.customer_id
order by customer.last_name;



------------------------------------------------------------------------------------------------------------------------------
--                                SECTION 7                                        --
------------------------------------------------------------------------------------------------------------------------------


--	7a. The music of Queen and Kris Kristofferson have seen an unlikely resurgence. As an unintended consequence, 
--	films starting with the letters `K` and `Q` have also soared in popularity. 
--	Use subqueries to display the titles of movies starting with the letters `K` and `Q` whose language is English.

select film.title
from film
where film.title like 'Q_%' or film.title like 'K_%' and film.language_id IN
(select distinct(language.language_id)
from film
join language on film.language_id = language.language_id
where language.name="English");

--	7b. Use subqueries to display all actors who appear in the film `Alone Trip`.
select actor.first_name, actor.last_name
from actor, film_actor
where actor.actor_id=film_actor.actor_id and film_actor.film_id IN
(select film.film_id
from film
where film.title = 'Alone Trip');


--	7c. You want to run an email marketing campaign in Canada, for which you will need the names and
--	email addresses of all Canadian customers. Use joins to retrieve this information.

select customer.first_name, customer.last_name, customer.email
from address, customer
where customer.address_id=address.address_id and address.city_id IN
(select city.city_id
from city
join country on country.country_id=city.country_id
where country='Canada');


--	7d. Sales have been lagging among young families, and you wish to target all family movies for a promotion. 
--	Identify all movies categorized as _family_ films.

select film.title as __family__films
from film
where film.film_id IN 
(select film_category.film_id
from category, film_category
where category.name="Family" and category.category_id=film_category.category_id);

--	7e. Display the most frequently rented movies in descending order.

select film.title,count(inventory.film_id) as rental_count
from rental
	join inventory on inventory.inventory_id=rental.inventory_id
    join film on film.film_id=inventory.film_id
group by inventory.film_id;

--	7f. Write a query to display how much business, in dollars, each store brought in.

select store.store_id as store,sum(payment.amount) as revenue
from staff, store, payment
where staff.store_id=store.store_id and payment.staff_id=staff.staff_id
group by store.store_id;

--	7g. Write a query to display for each store its store ID, city, and country.

select store.store_id, city.city, country.country 
from store
	join address on address.address_id=store.address_id
    join city on city.city_id=address.city_id
    join country on country.country_id=city.country_id;

--	7h. List the top five genres in gross revenue in descending order. (**Hint**: you may need to use the following tables: 
--	select category.name, sum(payment.amount) as revenue 

from payment
	join rental on rental.rental_id=payment.rental_id
    join inventory on inventory.inventory_id=rental.inventory_id
    join film_category on film_category.film_id=inventory.film_id
    join category on category.category_id=film_category.category_id
group by category.category_id
order by revenue desc
limit 5;


------------------------------------------------------------------------------------------------------------------------------
--                                SECTION 8                                        --
------------------------------------------------------------------------------------------------------------------------------
--	8a. In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross revenue.
--	Use the solution from the problem above to create a view. If you havent solved 7h, 
--	you can substitute another query to create a view.

create view Top_Grossing_FilmCategory as
select category.name, sum(payment.amount) as revenue 
from payment
	join rental on rental.rental_id=payment.rental_id
    join inventory on inventory.inventory_id=rental.inventory_id
    join film_category on film_category.film_id=inventory.film_id
    join category on category.category_id=film_category.category_id
group by catefilm_actorgory.category_id
order by revenue desc
limit 5;

--	8b. How would you display the view that you created in 8a?

select * from Top_Grossing_FilmCategory;

--	8c. You find that you no longer need the view `top_five_genres`. Write a query to delete it.

drop view Top_Grossing_FilmCategory;

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

## Uploading Homework

* To submit this homework using BootCampSpot:

  * Create a GitHub repository.
  * Upload your .sql file with the completed queries.
  * Submit a link to your GitHub repo through BootCampSpot.
