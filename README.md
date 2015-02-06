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
* Create a **Task** model.
	* verify it works in console 
* Setup a **`/tasks/`** index
	* iterate over each post
* Setup a **`/tasks/new`** and **CREATE**
	* Use form helpers with a new `Task`
* Setup a **`/tasks/:id`** to show a particular `tasks`

#### Part II: Setup Edit, Update, and Delete

* Setup a **`/tasks/:id/edit`** and **UPDATE**
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
### Create A Task 


In terminal, we create our `Task` model using a rails generator as follows,

	$ rails g model task content:string complete:boolean created_at:datetime
	$ rake db:migrate

#### Verify it works

We go straight into terminal to enter *rails console*.

	$ rails console

	> Task.create({content: "A task. this one is cool.", complete: false}) 
	=> #<Task ....>

*This will avoid issues later with `index` trying to render Creatures that aren't there.*

`db/seeds.rb`
	
	 Task.create({content: "This task sux", complete: false}) 
	 Task.create({content: "I'm really cool", description: false}) 



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



#### Task Index Route

Using the above routing pattern we'll write our first 
	
	`/config/routes.rb`

		RouteApp::Application.routes.draw do
			root to: 'site#index'
			get '/signup' to: 'site#signup'
			get '/login' to: 'site#login'
			get '/contact' to: 'site#contact'
			get '/about' to: 'site#about'
			
			# Also just to keep it RESTful
			get '/tasks', to: "tasks#index"
		end




#### Site Controller, Tasks Controller, and Index Method

Let's begin with the following 

	$ rails g controller site index signup login contact about
	
	$ rails g controller tasks

which is a generator for creating our controller.

We first need to setup our `#index` method in `site`

`app/controllers/site_controller.rb`

	class SiteController < ApplicationController
		
			def index

			end
		
		...
		
	end

	
Then setup our `#index` method in `tasks`

`app/controllers/tasks_controller.rb`

	class TasksController < ApplicationController
		
			def index
				@tasks = Task.all
				render :index
			end
		
		...
		
	end

#### Tasks Index View

If you look at your views the `views/tasks` folder has already be created so we just need to add the file below:

`app/views/tasks/index.html.erb`
	
	<% @tasks.each do |task| %>
		
		<div>
			Task: <%= task.content %> <br>
			Complete: <%=  task.complete %>
			Created At: <%=  task.created_at %>
		</div>
	
	<% end %>




### A new route for Tasks

The *RESTful* convention would be to make a form available at `/tasks/new`. Let's add this route.

`/config/routes.rb`

	Rails.application.routes.draw do
			root to: 'site#index'
			get '/signup' to: 'site#signup'
			get '/login' to: 'site#login'
			get '/contact' to: 'site#contact'
			get '/about' to: 'site#about'
		
		    # just to be RESTful
	 	    get '/tasks', to: 'tasks#index'
		    get '/tasks/new', to: 'tasks#new'
		
	end

### A new method for Tasks

The request for `/tasks/new` will search for a `tasks#new`, so we must create a method to handle this request. This will render the `new.html.erb` in the `app/views/tasks` folder.

`app/controllers/tasks_controller.rb`

	class TasksController < ApplicationController
		
		...
			def new
				render :new
			end
		
		...
		
	end



### A new view for Creatures

Let's create the `app/views/Tasks/new.html.erb` with a form that the user can use to sumbit new Tasks to the application. Note: the action is `/tasks` because it's the collection we are submiting to, and the method is `post` because we want to create.

`app/views/tasks/new.html.erb`

	<%= form_for :task, url: "/tasks", method: "post" do |f| %>
		
		<%= f.text_field :content %>
		<%= f.check_box :complete %>
		<%= f.submit "save task" %>
		
	<% end %>




### A Create Route

 We have now defined our next `route` in our `new.html.erb` as we are directing all form posts to the following:
	
	post "/tasks", to: "tasks#create"
	
when we said
	
	 url: "/tasks", method: "post"
	

and so we add it to our 

`/config/routes.rb`

	Rails.application.routes.draw do
		root to: 'site#index'
		get '/signup' to: 'site#signup'
		get '/login' to: 'site#login'
		get '/contact' to: 'site#contact'
		get '/about' to: 'site#about'
		
		# just to be RESTful
		get '/tasks', to: 'tasks#index'
		get '/tasks/new', to: 'tasks#new'
		post "/tasks", to: "tasks#create"
	end
	
#### A Create Method

Let's create `tasks#create` method 

`app/controllers/tasks_controller.rb`

	class TasksController < ApplicationController
		
		...
			def create
				new_task = params.require(:task).permit(:content, :complete)
				Task.create(new_task)
				redirect_to "/tasks"
			end
		
		...
		
	end

#### A smarter new view for Tasks


Let's update our `tasks#new` method 

`app/controllers/tasks_controller.rb`

	class TasksController < ApplicationController
		
		...
			def new
				@task = Task.new
				render :new
			end
		
		...
		
	end

This sets `@task` to a new instance of a `TAsk` which we can now share with or `new.html.erb` and thus our `form_helper`
	
`app/views/tasks/index.html.erb`
	
	<%= form_for @task do |f| %>
		
		<%= f.text_field :content %>
		<%= f.check_box :complete %>
		<%= f.submit "save task" %>
	
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
		get '/tasks', to: 'tasks#index'
		get '/tasks/new', to: 'tasks#new'
		# rake routes to check this route out
		get '/tasks/:id', to: 'tasks#show'
		post "/tasks", to: "tasks#create"
	end


Our `/tasks/:id` path is below our `/tasks/new` path. If we had `tasks/new` below the show route then the pattern matching will cause an error where all requests for `/tasks/new` get sent to the show.


A controller method  


`app/controllers/tasks_controller.rb`

	class TasksController < ApplicationController
		
		...
			def show
				id = params[:id]
				@task = Task.find(id)
				render :show
			end
		
		...
		
	end

A view for showing a task


`app/views/tasks/show.html.erb`

		<div>
			Content: <%= @task.content %> <br>
			Complete: <%=  @task.complete %> <br>
			Created At: <%= @task.created_at %>
		</div>
	

#### Changing the `#create` redirect

The `#create` method redirects to `#index` (the `/tasks` path), but this isn't very helpful for verrifying that a newly created task was properly created. The best way to fix this is to have it redirect to `#show`.


`app/controllers/tasks_controller.rb`

	class TasksController < ApplicationController
		
		...
			def create
				new_task = params.require(:task).permit(:content, :complete)
				task = Task.create(new_task)
				redirect_to "/tasks/#{task.id}"
			end
		
		...
		
	end


## Part II: Setup Edit, Update, and Delete

Editing a Task model requires two seperate methods. One to display the model information to be edited by the client, and another to handle updates submitted by the client.

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
			
		
			get '/tasks', to: 'tasks#index'
			
			get '/tasks/new', to: 'tasks#new'
			
			get '/tasks/:id', to: 'tasks#show'
				
			get '/tasks/:id/edit', to: 'tasks#edit'		
			post "/tasks", to: "tasks#create"
		end
		

* Similarly, using our `#show` method as inspiration we write an `#edit` method

	
	`app/controllers/tasks_controller.rb`
	
		class TasksController < ApplicationController
			
			...
				def edit
					id = params[:id]
					@task = Task.find(id)
					render :edit
				end
			
			...
			
		end


* Let's quickly begin the setup of an `edit` form using our `new.html.erb` from earlier. To see how the form is different we will need to render it and check it out in Chrome console.


	`app/views/tasks/new.html.erb`
	
		<%= form_for @task do |f| %>
			
			<%= f.text_field :content %>
			<%= f.check_box :complete %>
			<%= f.submit "update task" %>
			
		<% end %>

Going to [tasks/1/edit](localhost:3000/tasks/1/edit) we get the following error:
	
	undefined method `task_path' for #<#<Class:0x007fc5fc41be68>:0x007fc5fc40ea38>

This is because when we rake routes we notice that there is no `prefix` for the `task` which rails uses to internal generate methods for you.

`/config/routes.rb`

	Rails.application.routes.draw do
		root to: 'site#index'
		get '/signup' to: 'site#signup'
		get '/login' to: 'site#login'
		get '/contact' to: 'site#contact'
		get '/about' to: 'site#about'
		
		# Add prefixes to routes using `as: "some_prefix"` syntax
		get '/tasks', to: 'tasks#index', as: "tasks"
		
		get '/tasks/new', to: 'tasks#new', as: "new_task"
		
		get '/tasks/:id', to: 'tasks#show', as: "task"
			
		get '/tasks/:id/edit', to: 'tasks#edit', as: "edit_task"		
		post "/tasks", to: "tasks#create"
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
			get '/tasks', to: 'tasks#index', as: "tasks"
			
			get '/tasks/new', to: 'tasks#new', as: "new_task"
			
			get '/tasks/:id', to: 'tasks#show', as: "task"
				
			get '/tasks/:id/edit', to: 'tasks#edit', as: "edit_task"		
			
			post "/tasks", to: "tasks#create"
			
			# The update route
			patch "/tasks/:id", to: "tasks#update"
		end

		
	Note the method we now need to create is called `#update`
* In the `TasksController` we will create the `#update` method mentioned above
	
	`app/controllers/tasks_controller.rb`
	
		class TasksController < ApplicationController
			
			...
			
			def update
				task_id = params[:id]
				task = Task.find(task_id)
				
				# get updated data
				updated_attributes = params.require(:task).permit(:content, :complete)
				# update the creature
				task.update_attributes(updated_attributes)
				
				#redirect to show
				redirect_to "/tasks/#{task_id}"
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
			get '/tasks', to: 'tasks#index', as: "tasks"
			
			get '/tasks/new', to: 'tasks#new', as: "new_task"
			
			get '/tasks/:id', to: 'tasks#show', as: "task"
				
			get '/tasks/:id/edit', to: 'tasks#edit', as: "edit_task"		
			
			post "/tasks", to: "tasks#create"
			
			# The update route
			patch "/tasks/:id", to: "tasks#update"
			
			# the destroy route
			delete "/tasks/:id", to: "tasks#destroy"
		end
		

Next we create a method for it in the `TasksController`

`app/controllers/tasks_controller.rb`

	class TasksController < ApplicationController
		
		...
		
		def destroy
			id = params[:id]
			task = Task.find(id)
			task.destroy
			redirect_to "/tasks"
		end
		
	end
	
and if you were tempted to use [`Task.delete`](http://apidock.com/rails/ActiveRecord/Base/delete/class) that would be fine here because there are no associations. However, we need to use `model.destroy` if we want to avoid issues later.

Let's add a delete button to another view. 


`app/views/tasks/index.html.erb`
	
	<% @tasks.each do |task| %>
      <h2> <%= task.content %> </h2>
      <p> <%= task.complete %></p>
      <%= button_to "Delete", task, method: :delete %>
	<% end %>
