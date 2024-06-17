# msm-validations

Target: https://msm-gui.matchthetarget.com/

Lesson: https://learn.firstdraft.com/lessons/152-data-integrity-with-validations

<hr>

Goal: To be familiar with the validation feature in ActiveRecord to validate user-input data.

Notes:

1. First, let's break the app by going to views/movie_templates/show,html.erb and comment out the if-else statement and change it to just <%the_director.name%>, as follows.

```
<dt>
    Director
  </dt>
  <dd>

    <!--<% the_director = @the_movie.director %>

    <% if the_director != nil %>
      <%= the_director.name %>
    <% else %>
      Uh oh! We weren't able to find a director for this movie.
    <% end %>-->

  <%=the_director.name%>

  </dd>
```

2. Next, purposely destroy the director of one of the entries by first typing `rails console` in the terminal. Then, type the following in the terminal: `Director.where({name: "Francis Ford Coppola"}).at(0).destroy`.

3. As a result of the above action, when you visit the movies page and look at the netry for The Godfather, you will see the message in the director field, "Uh oh! We weren't able to find a director for this movie.".


#### Add validates function into the models classes

4. Having a blank entry for the director is unaceptable. To force user to enter a valid entery, we may use the validates function, as follows.

```
# app/models/movie.rb

class Movie < ApplicationRecord
  validates(:director_id, presence: true)

  def director
  # ...
end
```

5. Having added the validates function to the class as shown below, let's put it to test. The one-line function prevents the entry to be added if it doesn't contain information about the director_id.

```
class Movie < ApplicationRecord
  validates(:director_id, presence: true)
  
  def director
    my_director_id = self.director_id

    matching_directors = Director.where({ :id => my_director_id })
    
    the_director = matching_directors.at(0)

    return the_director
  end
endß
```

#### TEST 1

6. Let's create a new movie table without sepecifying the director_id, as follows:

- First, type `
```
m = Movie.new
```

Output:
```

[2] pry(main)> updated_at: nil> 
=> #<Movie:0x00007d70bb73e430 id: nil, title: nil, year: nil, duration: nil, description: nil, image: nil, director_id: nil, created_at: nil, updated_at: nil>
```

7. Next, try to save it to trigger the error.

```
m.save
```

The output is

```
=> false
```

8. Next, type m to get the following response:

```
 m
=> #<Movie:0x00007d70bb73e430 id: nil, title: nil, year: nil, duration: nil, description: nil, image: nil, director_id: nil, created_at: nil, updated_at: nil>ß
```
9. We can further probe into why the object didn't get save as follows:

- Type m.errors]
```
m.errors
=> #<ActiveModel::Errors [#<ActiveModel::Error attribute=director_id, type=blank, options={}>]>
```

- Type m.errors.messages:

```
m.errors.messages
=> {:director_id=>["can't be blank"]}
```

9. Type `m.errors.full_messages`:

```
m.errors.full_messages
=> ["Director can't be blank"]
```

#### Test 2

10. Let's ad another kind of validator called `validates(:title, uniqueness: true)`. This means that the user must enter a unique title.s

Here is the new code:

```
# app/models/movie.rb

class Movie < ApplicationRecord
  validates(:director_id, presence: true)
  validates(:title, uniqueness: true)

  def director
  # ...
end
```

11. Let's create two entries. One will be successful and the other will not be.

-  Successful example

```
[11] pry(main)> m = Movie.new
[12] pry(main)> m.title = "zebra"
[13] pry(main)> m.director_id = 1
[14] pry(main)> m.save
=> true
```

- Unsuccessful example 

Saving a new object that share the same title, "zebra" as m and having no director_id failed in both instances.

```
[15] pry(main)> n = Movie.new
[16] pry(main)> n.title = "zebra"
[17] pry(main)> n.save
   (0.1ms)  begin transaction
  Movie Exists? (0.2ms)  SELECT 1 AS one FROM "movies" WHERE "movies"."title" = ? LIMIT ?  [["title", "zebra"], ["LIMIT", 1]]
   (0.0ms)  rollback transaction
=> false
[18] pry(main)> n.errors.full_messages
=> ["Director can't be blank", "Title has already been taken"]
```
***
