# Forms and Resources
## More CRUD


## Intro

| Objectives |
| :---- |
| Review **CRUD** in the context of a Rails application, especially **Updating** and **Deleting** a resource. |
| Examine **form helpers** and **partials** (if time permits) in a  Rails Application. |
| Apply styling and **Bootstrap** to our site to create a custom layout. |


### Previously

* Routing requests to a controller method
	* **INDEX**
	* **NEW** 
	* **CREATE**
* RESTful resources
* Basic HTML Forms 	


#### Part I: Review and Apply Form Helpers

* Drop in Bootstrap
* Create a **Todo** model.
	* verify it works in console 
* Setup a **`/todo/`** index
	* iterate over each post
* Setup a **`/todo/new`** and **CREATE**
	* Use form helpers with a new `Todo`
* Setup a **`/todo/:id`** to show a particular `todo`

#### Part II: Setup Edit, Update, and Delete

* Setup a **`/todo/:id/edit`** and **UPDATE**
* Setup a **Delete**


## Part I: Review and Refactor


### CRUD and REST Reference

Typically we associate **CRUD** with the following **HTTP** methods

| CRUD Operation | HTTP Method | Example|
| :---  |	:--- | :-- |
| Create | POST | `POST "/puppies?name=spot"` (create a puppy named spot) |
| Read   | GET  | `GET "/puppies"` (Shows all puppies) |
| Update | PUT or UPDATE | `PUT "/puppies/1?name=lassy"` (change puppy number 1 to have name lassy) |
| Delete | DELETE | `Delete "/puppies/1"` (destroy the first puppy, yikes!!!!) |

REST stands for **REpresentational State Transfer**. We will demonstrate these practices throughout this lesson, but for now preparing don't worry too much about it yet.




### Setup Rails New


Make sure you have your `psql` setup using `which psql` if nothing shows up then do the following:

```
echo 'export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/9.3/bin/' >> ~/.bash_profile && source ~/.bash_profile
```


* `$ rails new bog_app -T -d postgresql`
* `$ cd bog_app`
*  NOTE: `rake db:create`
* `$ rails s`

Now our app is up and running, [localhost:3000](localhost:3000/). 

### Drop in Bootstrap

Just put the third party css libraries in `vendor/assets` and for bootstrap just file it under stylesheets.

```
 curl https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css > vendor/assets/stylesheets/bootstrap-3.2.0.min.css
```
### Create A Todo 


In terminal, we create our `Todo` model using a rails generator as follows,

	$ rails g model Todo content:string complete:boolean created_at:datetime
	$ rake db:migrate

#### Verify it works

We go straight into terminal to enter *rails console*.

	$ rails console

	> Todo.create({content: "A todo. this one is cool.", complete: false}) 
	=> #<Todo ....>

*This will avoid issues later with `index` trying to render Creatures that aren't there.*

`db/seeds.rb`
	
	 Todo.create({content: "This todo sux", complete: false}) 
	 Todo.create({content: "I'm really cool", description: false}) 



 and then just run `$ rake db:seed` in console. This will now get run every time you `rake db:reset`.
 
#### Seeds

Because when we create an application in development we typically will want some mock data to play with we can just drop this into the `db/seeds.rb` file.



### Routes


Go to `config/routes.rb` and inside the routes block erase all the commented text. It should now look exactly as follows

`config/routes.rb`

	Rails.application.routes.draw do

	end




Now we can define all our routes.

Your `routes.rb` will just be telling your app how to connect *HTTP* requests to a **Controller**. Let's get ready for our first route. 

> NOTE
> 
> * The nature of any route goes as follows:
> 
>		request_type '/for/some/path/goes', to: "controller#method"
>
>	e.g. if we had a `PuppiesController` that had a `index` method we could say
>
>		get "/puppies", to: "puppies#index"



#### Todo Index Route

Using the above routing pattern we'll write our first 
	
	`/config/routes.rb`

		RouteApp::Application.routes.draw do
			root to: 'site#index'
			get '/signup' to: 'site#signup'
			get '/login' to: 'site#login'
			get '/contact' to: 'site#contact'
			get '/about' to: 'site#about'
			
			# Also just to keep it RESTful
			get '/todos', to: "todos#index"
		end




#### Site Controller, Todos Controller, and Index Method

Let's begin with the following 

	$ rails g controller site index signup login contact about
	
	$ rails g controller todos

which is a generator for creating our controller.

We first need to setup our `#index` method in `site`

`app/controllers/site_controller.rb`

	class SiteController < ApplicationController
		
			def index

			end
		
		...
		
	end

	
Then setup our `#index` method in `todos`

`app/controllers/todos_controller.rb`

	class TodosController < ApplicationController
		
			def index
				@todos = Todo.all
				render :index
			end
		
		...
		
	end

#### Todos Index View

If you look at your views the `views/todos` folder has already be created so we just need to add the file below:

`app/views/todos/index.html.erb`
	
	<% @todos.each do |todo| %>
		
		<div>
			Todo: <%= Todo.content %> <br>
			Complete: <%=  Todo.complete %>
			Created At: <%=  Todo.created_at %>
		</div>
	
	<% end %>




### A new route for Todos

The *RESTful* convention would be to make a form available at `/todos/new`. Let's add this route.

`/config/routes.rb`

	Rails.application.routes.draw do
			root to: 'site#index'
			get '/signup' to: 'site#signup'
			get '/login' to: 'site#login'
			get '/contact' to: 'site#contact'
			get '/about' to: 'site#about'
		
		    # just to be RESTful
	 	    get '/todos', to: 'todos#index'
		    get '/todos/new', to: 'todos#new'
		
	end

### A new method for Todos

The request for `/Todos/new` will search for a `Todos#new`, so we must create a method to handle this request. This will render the `new.html.erb` in the `app/views/Todos` folder.

`app/controllers/todos_controller.rb`

	class TodosController < ApplicationController
		
		...
			def new
				render :new
			end
		
		...
		
	end



### A new view for Creatures

Let's create the `app/views/Todos/new.html.erb` with a form that the user can use to sumbit new Todos to the application. Note: the action is `/Todos` because it's the collection we are submiting to, and the method is `post` because we want to create.

`app/views/todos/new.html.erb`

	<%= form_for :todo, url: "/todos", method: "post" do |f| %>
		
		<%= f.text_field :content %>
		<%= f.check_box :complete %>
		<%= f.submit "save todo" %>
		
	<% end %>




### A Create Route

 We have now defined our next `route` in our `new.html.erb` as we are directing all form posts to the following:
	
	post "/todos", to: "todos#create"
	
when we said
	
	 url: "/todos", method: "post"
	

and so we add it to our 

`/config/routes.rb`

	Rails.application.routes.draw do
		root to: 'site#index'
		get '/signup' to: 'site#signup'
		get '/login' to: 'site#login'
		get '/contact' to: 'site#contact'
		get '/about' to: 'site#about'
		
		# just to be RESTful
		get '/todos', to: 'todos#index'
		get '/todos/new', to: 'todos#new'
		post "/todos", to: "todos#create"
	end
	
#### A Create Method

Let's create `todos#create` method 

`app/controllers/todos_controller.rb`

	class TodosController < ApplicationController
		
		...
			def create
				new_todo = params.require(:todo).permit(:content, :complete)
				Todo.create(new_todo)
				redirect_to "/todos"
			end
		
		...
		
	end

#### A smarter new view for Todos


Let's update our `todos#new` method 

`app/controllers/todos_controller.rb`

	class TodosController < ApplicationController
		
		...
			def new
				@todo = Todo.new
				render :new
			end
		
		...
		
	end

This sets `@todo` to a new instance of a `Todo` which we can now share with or `new.html.erb` and thus our `form_helper`
	
`app/views/todos/index.html.erb`
	
	<%= form_for @todo do |f| %>
		
		<%= f.text_field :content %>
		<%= f.check_box :complete %>
		<%= f.submit "save todo" %>
	
	<% end %>




### Show Route

Right now, our app redirects to  `#index` after a create, which isn't helpful for quickly verifying what you just created. To do this we create a `#show`.

Let's add our `show` route.
	
`/config/routes.rb`

	Rails.application.routes.draw do
		root to: 'site#index'
		get '/signup' to: 'site#signup'
		get '/login' to: 'site#login'
		get '/contact' to: 'site#contact'
		get '/about' to: 'site#about'
		
		# just to be RESTful
		get '/todos', to: 'todos#index'
		get '/todos/new', to: 'todos#new'
		# rake routes to check this route out
		get '/todos/:id', to: 'todos#show'
		post "/todos", to: "todos#create"
	end


Our `/todos/:id` path is below our `/todos/new` path. If we had `todos/new` below the show route then the pattern matching will cause an error where all requests for `/todos/new` get sent to the show.


A controller method  


`app/controllers/todos_controller.rb`

	class TodosController < ApplicationController
		
		...
			def show
				id = params[:id]
				@todo = Todo.find(id)
				render :show
			end
		
		...
		
	end

A view for showing a todo


`app/views/todos/show.html.erb`

		<div>
			Content: <%= @todo.content %> <br>
			Complete: <%=  @todo.complete %> <br>
			Created At: <%= @todo.created_at %>
		</div>
	

#### Changing the `#create` redirect

The `#create` method redirects to `#index` (the `/todos` path), but this isn't very helpful for verrifying that a newly created todo was properly created. The best way to fix this is to have it redirect to `#show`.


`app/controllers/todos_controller.rb`

	class TodosController < ApplicationController
		
		...
			def create
				new_todo = params.require(:todo).permit(:content, :complete)
				todo = Todo.create(new_todo)
				redirect_to "/todos/#{todo.id}"
			end
		
		...
		
	end


## Part II: Setup Edit, Update, and Delete

Editing a Todo model requires two seperate methods. One to display the model information to be edited by the client, and another to handle updates submitted by the client.

If look back at how we handled the getting of our `new` form we see the following pattern.

* Make a route first
* Define a controller method 	
* render view

The only difference is that now we need to use the `id` of the object to be edited. We get the following battle plan.

* Make a route first
	* Make sure it specifies the `id` of the thing to be edited
* Define a controller method 
	* Retrieve the `id` of the model to be edited from `params`
	* use the `id` to find the model
* render view
	* use model to display in the form 

### Getting to an Edit

We begin with handling the request from a client for an edit page. 

* We can easily define a **RESTful** route to handle getting the edit page as follows

	`/config/routes.rb`
	
		Rails.application.routes.draw do
			root to: 'site#index'
			get '/signup' to: 'site#signup'
			get '/login' to: 'site#login'
			get '/contact' to: 'site#contact'
			get '/about' to: 'site#about'
			
		
			get '/todos', to: 'todos#index'
			
			get '/todos/new', to: 'todos#new'
			
			get '/todos/:id', to: 'todos#show'
				
			get '/todos/:id/edit', to: 'todos#edit'		
			post "/todos", to: "todos#create"
		end
		

* Similarly, using our `#show` method as inspiration we write an `#edit` method

	
	`app/controllers/todoss_controller.rb`
	
		class TodossController < ApplicationController
			
			...
				def edit
					id = params[:id]
					@todo = Todo.find(id)
					render :edit
				end
			
			...
			
		end


* Let's quickly begin the setup of an `edit` form using our `new.html.erb` from earlier. To see how the form is different we will need to render it and check it out in Chrome console.


	`app/views/todos/new.html.erb`
	
		<%= form_for @todo do |f| %>
			
			<%= f.text_field :content %>
			<%= f.check_box :complete %>
			<%= f.submit "update todo" %>
			
		<% end %>

Going to [todos/1/edit](localhost:3000/todos/1/edit) we get the following error:
	
	undefined method `todo_path' for #<#<Class:0x007fc5fc41be68>:0x007fc5fc40ea38>

This is because when we rake routes we notice that there is no `prefix` for the `todo` which rails uses to internal generate methods for you.

`/config/routes.rb`

	Rails.application.routes.draw do
		root to: 'site#index'
		get '/signup' to: 'site#signup'
		get '/login' to: 'site#login'
		get '/contact' to: 'site#contact'
		get '/about' to: 'site#about'
		
		# Add prefixes to routes using `as: "some_prefix"` syntax
		get '/todos', to: 'todos#index', as: "todos"
		
		get '/todos/new', to: 'todos#new', as: "new_todo"
		
		get '/todos/:id', to: 'todos#show', as: "todo"
			
		get '/todos/:id/edit', to: 'todos#edit', as: "edit_todo"		
		post "/todos", to: "todos#create"
	end


That's pretty much the whole-shebang when comes to getting an edit page. Our previous knowledge has really come to help us understand what we need to do. We'll see this also true for the update that still needs to be handled witht the submission of the form above.


### Putting updated form data 

Looking back at how we handled the submission of our `new` form we see the following pattern.

* Make a route first
* Define a controller method 
* redirect to something

The only difference now is that we will need to use the `id` of the object being update.

* Make a route first
	* Make sure it specifies the `id` of the thing to be **updated**
* Define a controller method 
	* Retrieve the `id` of the model to be **updated** from `params`
	* use the `id` to find the model
	* retrieve the updated info sent from the form in `params`
	* update the model
* redirect to show
	* use `id` to redirect to `#show` 
	
### Putting it into action

* **Make a route** that uses the `id` of the object to be updated

	`/config/routes.rb`
	
		Rails.application.routes.draw do
			root to: 'site#index'
			get '/signup' to: 'site#signup'
			get '/login' to: 'site#login'
			get '/contact' to: 'site#contact'
			get '/about' to: 'site#about'
			
			# Add prefixes to routes using `as: "some_prefix"` syntax
			get '/todos', to: 'todos#index', as: "todos"
			
			get '/todos/new', to: 'todos#new', as: "new_todo"
			
			get '/todos/:id', to: 'todos#show', as: "todo"
				
			get '/todos/:id/edit', to: 'todos#edit', as: "edit_todo"		
			
			post "/todos", to: "todos#create"
			
			# The update route
			patch "/todos/:id", to: "todos#update"
		end

		
	Note the method we now need to create is called `#update`
* In the `TodossController` we will create the `#update` method mentioned above
	
	`app/controllers/todos_controller.rb`
	
		class TodosController < ApplicationController
			
			...
			
			def update
				todo_id = params[:id]
				todo = Todo.find(todo_id)
				
				# get updated data
				updated_attributes = params.require(:todo).permit(:content, :complete)
				# update the creature
				todo.update_attributes(updated_attributes)
				
				#redirect to show
				redirect_to "/todos/#{todo_id}"
			end
			
		end

### Destroy


Following a similar pattern to the above we create a route for a destroy that uses the `id` of the model to be deleted.

	`/config/routes.rb`
	
		Rails.application.routes.draw do
			root to: 'site#index'
			get '/signup' to: 'site#signup'
			get '/login' to: 'site#login'
			get '/contact' to: 'site#contact'
			get '/about' to: 'site#about'
			
			# Add prefixes to routes using `as: "some_prefix"` syntax
			get '/todos', to: 'todos#index', as: "todos"
			
			get '/todos/new', to: 'todos#new', as: "new_todo"
			
			get '/todos/:id', to: 'todo#show', as: "todo"
				
			get '/todos/:id/edit', to: 'todo#edit', as: "edit_todo"		
			
			post "/todos", to: "todos#create"
			
			# The update route
			patch "/todos/:id", to: "todos#update"
			
			# the destroy route
			delete "/todos/:id", to: "todos#destroy"
		end
		

Next we create a method for it in the `TodossController`

`app/controllers/Todos_controller.rb`

	class TodosController < ApplicationController
		
		...
		
		def destroy
			id = params[:id]
			todo = Todo.find(id)
			todo.destroy
			redirect_to "/todos"
		end
		
	end
	
and if you were tempted to use [`Todo.delete`](http://apidock.com/rails/ActiveRecord/Base/delete/class) that would be fine here because there are no associations. However, we need to use `model.destroy` if we want to avoid issues later.

Let's add a delete button to another view. 


`app/views/todos/index.html.erb`
	
	<% @todos.each do |todo| %>
      <h2> <%= todo.content %> </h2>
      <p> <%= todo.complete %></p>
      <%= button_to "Delete", todo, method: :delete %>
	<% end %>
