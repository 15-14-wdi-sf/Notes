##Before we get started
Let's modify our `new` method in our sessions controller

`sessions_controller.rb`

```
class SessionsController < ApplicationController
  def new
    if current_user
      redirect_to user_path(current_user.id)
    end
  end
  
  ...
 
end
```

##Respond to json
Add a respond_to template like

```
	respond_to do |format|
      	format.html { code_here }
      	format.json { render json: @tasks }
    end
```

to the `index`, `create`, `show`, `update`, and `destroy` methods

`tasks_controller.rb`

```
class TasksController < ApplicationController

	before_action :get_user

	def index
		@tasks = @user.tasks.all
		#render :index

		respond_to do |format|
      		format.html
      		format.json { render json: @tasks }
    	end
	end

	def new
		@task = @user.tasks.new
		render :new
	end

	def create
		new_task = params.require(:task).permit(:content, :complete)
		@task = @user.tasks.create(new_task)
		respond_to do |format|
      		format.html { redirect_to "/users/#{@user.id}/tasks/#{@task.id}" }
      		format.json { render json: @task }
    	end
	end

	def show
		task_id = params[:task_id]
		@task = @user.tasks.find(task_id)
		#@task = Task.find(task_id)

		respond_to do |format|
      		format.html
      		format.json { render json: @task }
    	end
	end

	def edit
		task_id = params[:task_id]
		@task = @user.tasks.find(task_id)
		render :edit
	end

	def update
		task_id = params[:task_id]
		@task = @user.tasks.find(task_id)
		updated_attrs = params.require(:task).permit(:content, :complete)
		@task.update_attributes(updated_attrs)
		

		respond_to do |format|
      		format.html { redirect_to task_path }
      		format.json { render json: @task }
    	end

	end

	def destroy
		task_id = params[:task_id]
		task = @user.tasks.find(task_id)
		task.destroy

		respond_to do |format|
      		format.html { redirect_to tasks_path }
      		format.json { render json: @task }
    	end	
	end

	private

		def get_user
			if params[:user_id]
				user_id = params[:user_id]
				@user = User.find(user_id)
			else
				task_id = params[:nested_task_id]
				@nested_task = Task.find(task_id)
			end
		end

end
```


Sweet. Now lets get started, with the rest of the lesson.

##Ajax and jQuery
**NOTE: If you haven't yet, you should go into your `rails console` and try to make a `todo` so that it can show up...**

Go to [`localhost:3000/users/1/tasks.json`](localhost:3000/users/1/todos.json) and verify this is working correctly as a `JSON` endpoint.


We also should make our `users/index.html.erb` have a `todos-con` to hold our `tasks`, like `<div id="tasks-con"> </div>`

In the `application.js` we need to add our JS for loading all the `tasks`.

```
$(function () {
	var userId = window.location.pathname.split('/')[2];
	var usersTasksURL = "/users/"+ userId + "/tasks.json";
	
    $.get(usersTasksURL)
      .done(function (todos) {
        console.log("All Todos:", todos);
      });
});
```

Then we want to use `jQuery` to `append` the `task` the page.

```
$(function () {
	var userId = window.location.pathname.split('/')[2];
	var usersTasksURL = "/users/"+ userId + "/tasks.json";

	var $tasksCon = $("#tasks-con");

    $.get(usersTasksURL)
      .done(function (tasks) {
      	tasks.forEach(function(task){
      		var $task = $("<div>" + task.content + "</div>");
      		console.log(task);
      		$tasksCon.append($task);
      	});
        console.log("All Tasks:", tasks);
      });
});

```

### Creating 

Let's modify our `users/show.html.erb` to have a form for a `tasks`.

```
<form id="task-form">
	<input id="content" type="text" placeholder="Todo" name="content"/>
	<button id="task-button">Add Todo</button>
</form>
```

Note that if you view page on the `localhost:3000` you should see that the form has `new_todo` as an id.
Editing

Let's wait for the submit event from the form. Then we should `preventDefault()` on the form.


```
  var $taskForm = $("#task-form");

  $taskForm.on("submit", function (event) {
    event.preventDefault();
    console.log($(this).serialize());
  });

```

All in all we have the following:

```
$(function () {
	var userId = window.location.pathname.split('/')[2];
	var usersTasksURL = "/users/"+ userId + "/tasks.json";

	var $tasksCon = $("#tasks-con");

    $.get(usersTasksURL)
      .done(function (tasks) {
      	tasks.forEach(function(task){
      		var $task = $("<div>" + task.content + "</div>");
      		console.log(task);
      		$tasksCon.append($task);
      	});
        console.log("All Todos:", tasks);
      });


	var $taskForm = $("#task-form");
	var $taskContent = $("#content");

	$taskForm.on("submit", function (event) {
		event.preventDefault();
		
		var content = $taskContent.val();
		var obj = {task: {content: content}};
		
		$.post(usersTasksURL, obj, function(task){
			console.log(task);
		});
	});
});

```


Now that our form is submitted, we can add the in the logic to `post` the data to our backend and render it to the page.



```
$taskForm.on("submit", function (event) {
		event.preventDefault();
		var content = $taskContent[0].value;
		console.log(content);
		var obj = {task: {content: content}};
		$.post(usersTasksURL, obj, function(task){
			console.log(task);

			var $task = $("<div>" + task.content + "</div>");
      		$tasksCon.append($task);
      		$taskContent[0].value = "";
		});
	});
```


## Part 2: Deleting and Editing

In order to setup a delete we need to add `delete` buttons to all our elements, and the listen for clicks from any of those buttons.


Let's update our `append` to have a button.


```
$(function () {
	var userId = window.location.pathname.split('/')[2];
	var usersTasksURL = "/users/"+ userId + "/tasks.json";

	var $tasksCon = $("#tasks-con");

    $.get(usersTasksURL)
      .done(function (tasks) {
      	tasks.forEach(function(task){
      		var taskClass = 'class="task-' + task.id + '"';
      		var $taskContent = $("<div " + taskClass + ">" + task.content + "</div>");
      		var $taskDelete = $("<button class='delete'>Delete</button>");
      		var $task = $taskContent.append($taskDelete);
      		console.log(task);
      		$tasksCon.append($task);
      	});
        console.log("All Todos:", tasks);
      });


	var $taskForm = $("#task-form");
	var $taskContent = $("#content");

	$taskForm.on("submit", function (event) {
		event.preventDefault();
		var content = $taskContent[0].value;
		console.log(content);
		var obj = {task: {content: content}};
		$.post(usersTasksURL, obj, function(task){
			console.log(task);
			var taskClass = 'class="task-' + task.id + '"';
      		var $taskContent = $("<div " + taskClass + ">" + task.content + "</div>");
      		var $taskDelete = $("<button class='delete'>Delete</button>");
      		var $task = $taskContent.append($taskDelete);
      		$tasksCon.append($task);
      		$taskContent[0].value = "";
		});
	});
});

```




##Allowing Deletes
The main functionality is going to be in a new function called `watchForDeletes`, which attatches an event listener to our delete buttons.

```
	var watchForDeletes = function(){
		var $deletes = $('.delete');

		$deletes.click(function(event){
			console.log(event);
			event.preventDefault();
			var $task = event.toElement.parentElement;
			var taskId = $task.classList[0].split("-")[1];
			var usersTaskURL = "/users/"+ userId + "/tasks/" + taskId + ".json";
			console.log(usersTaskURL);

			var deleteObj = {method: "DELETE", url: usersTaskURL};	

			$.ajax(deleteObj).done(function(task){
				console.log(task);
				$task.remove();
			});
		});
	};
```

Added to the rest of our code base, we get


```
$(function () {
	var userId = window.location.pathname.split('/')[2];
	var usersTasksURL = "/users/"+ userId + "/tasks.json";

	var $tasksCon = $("#tasks-con");

    $.get(usersTasksURL)
      .done(function (tasks) {
      	tasks.forEach(function(task){
      		var taskClass = 'class="task-' + task.id + '"';
      		var $taskContent = $("<div " + taskClass + ">" + task.content + "</div>");
      		var $taskDelete = $("<button class='delete'>Delete</button>");
      		var $task = $taskContent.append($taskDelete);
      		console.log(task);
      		$tasksCon.append($task);
      	});
      	watchForDeletes();
        console.log("All Todos:", tasks);
      });


	var $taskForm = $("#task-form");
	var $taskContent = $("#content");

	$taskForm.on("submit", function (event) {
		event.preventDefault();
		var content = $taskContent[0].value;
		console.log(content);
		var obj = {task: {content: content}};
		$.post(usersTasksURL, obj, function(task){
			console.log(task);
			var taskClass = 'class="task-' + task.id + '"';
      		var $taskContent = $("<div " + taskClass + ">" + task.content + "</div>");
      		var $taskDelete = $("<button class='delete'>Delete</button>");
      		var $task = $taskContent.append($taskDelete);
      		$tasksCon.append($task);
      		$taskContent[0].value = "";
      		watchForDeletes();
		});
    	//console.log($(this).serialize());
	});

	var watchForDeletes = function(){
		var $deletes = $('.delete');

		$deletes.click(function(event){
			console.log(event);
			event.preventDefault();
			var $task = event.toElement.parentElement;
			var taskId = $task.classList[0].split("-")[1];
			var usersTaskURL = "/users/"+ userId + "/tasks/" + taskId + ".json";
			console.log(usersTaskURL);

			var deleteObj = {method: "DELETE", url: usersTaskURL};	

			$.ajax(deleteObj).done(function(task){
				console.log(task);
				$task.remove();
			});
		});
	};
});
```


## Updates and Editing


### Checkboxes

One of the simplest updates we can do is to add a completed `checkbox`. Let's update our `append` to also have the `checkbox`.

```
var taskClass = 'class="task-' + task.id + '"';
var $taskContent = $("<div " + taskClass + ">" + task.content + "</div>");
var isChecked = task.complete ? "checked" : "";
var $taskComplete = $("<input type='checkbox' class='complete'" + isChecked + "/>");
var $taskDelete = $("<button class='delete'>Delete</button>");
var $task = $taskContent.append($taskComplete).append($taskDelete);
$tasksCon.append($task);
```

Now lets update the tasks accordingly. Which should result in the following.

```
$(function () {
	var userId = window.location.pathname.split('/')[2];
	var usersTasksURL = "/users/"+ userId + "/tasks.json";

	var $tasksCon = $("#tasks-con");

    $.get(usersTasksURL)
      .done(function (tasks) {
      	tasks.forEach(function(task){
      		var taskClass = 'class="task-' + task.id + '"';
      		var $taskContent = $("<div " + taskClass + ">" + task.content + "</div>");
      		var isChecked = task.complete ? "checked" : "";
			var $taskComplete = $("<input type='checkbox' class='complete'" + isChecked + "/>");
      		var $taskDelete = $("<button class='delete'>Delete</button>");
      		var $task = $taskContent.append($taskComplete).append($taskDelete);
      		console.log(task);
      		$tasksCon.append($task);
      	});
      	watchForDeletes();
        console.log("All Todos:", tasks);
      });


	var $taskForm = $("#task-form");
	var $taskContent = $("#content");

	$taskForm.on("submit", function (event) {
		event.preventDefault();
		var content = $taskContent[0].value;
		console.log(content);
		var obj = {task: {content: content}};
		$.post(usersTasksURL, obj, function(task){
			console.log(task);
			var taskClass = 'class="task-' + task.id + '"';
      		var $taskContent = $("<div " + taskClass + ">" + task.content + "</div>");
      		var isChecked = task.complete ? "checked" : "";
            var $taskComplete = $("<input type='checkbox' class='complete'" + isChecked + "/>");
      		var $taskDelete = $("<button class='delete'>Delete</button>");
      		var $task = $taskContent.append($taskComplete).append($taskDelete);
      		$tasksCon.append($task);
      		$taskContent[0].value = "";
      		watchForDeletes();
		});
    	//console.log($(this).serialize());
	});

	var watchForDeletes = function(){
		var $deletes = $('.delete');

		$deletes.click(function(event){
			console.log(event);
			event.preventDefault();
			var $task = event.toElement.parentElement;
			var taskId = $task.classList[0].split("-")[1];
			var usersTaskURL = "/users/"+ userId + "/tasks/" + taskId + ".json";
			console.log(usersTaskURL);

			var deleteObj = {method: "DELETE", url: usersTaskURL};	

			$.ajax(deleteObj).done(function(task){
				console.log(task);
				$task.remove();
			});
		});
	};
});
```

###Now we need to watch for clicks on the checkbox

Here's a method for that

```
var watchForTaskCompleteUpdates = function() {
		var $completes = $('.complete');

		$completes.click(function(event){
			console.log(event);
			var $checkbox = event.toElement;
			var $task = $checkbox.parentElement;
			var taskId = $task.classList[0].split("-")[1];
			var usersTaskURL = "/users/"+ userId + "/tasks/" + taskId + ".json";
			var checked = $checkbox.checked;

			var patchObj = {method: "PATCH", url: usersTaskURL, data: {task: {complete: checked}}};
			$.ajax(patchObj).done(function(task){
				console.log(task);
			});
		});
	};
```

and the fully done version here:

```
// This is a manifest file that'll be compiled into application.js, which will include all the files
// listed below.
//
// Any JavaScript/Coffee file within this directory, lib/assets/javascripts, vendor/assets/javascripts,
// or any plugin's vendor/assets/javascripts directory can be referenced here using a relative path.
//
// It's not advisable to add code directly here, but if you do, it'll appear at the bottom of the
// compiled file.
//
// Read Sprockets README (https://github.com/sstephenson/sprockets#sprockets-directives) for details
// about supported directives.
//
//= require jquery
//= require jquery_ujs
//= require turbolinks
//= require_tree .
// wait for window load
$(function () {
	var userId = window.location.pathname.split('/')[2];
	var usersTasksURL = "/users/"+ userId + "/tasks.json";

	var $tasksCon = $("#tasks-con");

    $.get(usersTasksURL)
      .done(function (tasks) {
      	tasks.forEach(function(task){
      		var taskClass = 'class="task-' + task.id + '"';
      		var $taskContent = $("<div " + taskClass + ">" + task.content + "</div>");
      		var isChecked = task.complete ? "checked" : "";
      		var $taskComplete = $("<input type='checkbox' class='complete'" + isChecked + "/>");
      		var $taskDelete = $("<button class='delete'>Delete</button>");
      		var $task = $taskContent.append($taskComplete).append($taskDelete);
      		console.log(task);
      		$tasksCon.append($task);
      	});
      	watchForDeletes();
      	watchForTaskCompleteUpdates();
        console.log("All Todos:", tasks);
      });


	var $taskForm = $("#task-form");
	var $taskContent = $("#content");

	$taskForm.on("submit", function (event) {
		event.preventDefault();
		var content = $taskContent[0].value;
		console.log(content);
		var obj = {task: {content: content}};
		$.post(usersTasksURL, obj, function(task){
			console.log(task);
			var taskClass = 'class="task-' + task.id + '"';
      		var $taskContent = $("<div " + taskClass + ">" + task.content + "</div>");
      		var isChecked = task.complete ? "checked" : "";
      		var $taskComplete = $("<input type='checkbox' class='complete'" + isChecked + "/>");
      		var $taskDelete = $("<button class='delete'>Delete</button>");
      		var $task = $taskContent.append($taskComplete).append($taskDelete);
      		$tasksCon.append($task);
      		$taskContent[0].value = "";
      		watchForDeletes();
      		watchForTaskCompleteUpdates();
		});
    	//console.log($(this).serialize());
	});

	var watchForDeletes = function(){
		var $deletes = $('.delete');

		$deletes.click(function(event){
			console.log(event);
			event.preventDefault();
			var $task = event.toElement.parentElement;
			var taskId = $task.classList[0].split("-")[1];
			var usersTaskURL = "/users/"+ userId + "/tasks/" + taskId + ".json";
			console.log(usersTaskURL);

			var deleteObj = {method: "DELETE", url: usersTaskURL};	

			$.ajax(deleteObj).done(function(task){
				console.log(task);
				$task.remove();
			});
		});
	};

	var watchForTaskCompleteUpdates = function() {
		var $completes = $('.complete');

		$completes.click(function(event){
			console.log(event);
			var $checkbox = event.toElement;
			var $task = $checkbox.parentElement;
			var taskId = $task.classList[0].split("-")[1];
			var usersTaskURL = "/users/"+ userId + "/tasks/" + taskId + ".json";
			var checked = $checkbox.checked;

			var patchObj = {method: "PATCH", url: usersTaskURL, data: {task: {complete: checked}}};
			$.ajax(patchObj).done(function(task){
				console.log(task);
			});
		});
	};
});
```

##Let's hope we get to here