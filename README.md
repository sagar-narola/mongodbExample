Mongodb

Ref : https://docs.mongodb.com/mongoid/master/#ruby-mongoid-tutorial

Ref :  http://kerrizor.com/blog/2014/04/02/quick-intro-to-mongodb-in-rails

Ref :  http://railscasts.com/episodes/238-mongoid?view=asciicast

Ref :  https://richonrails.com/articles/mongodb-and-rails


1. install mongodb

    brew install mongodb

2. update mondodb

  	brew upgrade mongodb

3. start mongodb

  	brew services start mongodb

4. Check mongodb start or not

  	http://localhost:27017/

  	(O/P : It looks like you are trying to access MongoDB over HTTP on the native driver port.)

    4.1 stop mongodb

  	brew services stop mongodb

5. open mongo console

	 mongo

6. create new project

	 rails new mongo-people --skip-active-record

7. gem

	 gem "mongoid”

8. bundle install

	 bundle install

9. Generate mongodb config file

  	rails g mongoid:config

  	(Note : Just as when using a relational DB such as SQLite, Postgres, or MySQL, we need a configuration file. Mongoid installs a custom Rails generator for us)


10. create the model (Now we are ready to create model)

  	rails g model person first_name last_name email notes

  	(Note :  Instead of inheriting from ActiveRecord::Base, the first line in the model, include Mongoid::Document, tells rails that we wish to use Mongoid and interact with MongoDB. The second, third, fourth, and fifth lines in the model all define the various fields. MongoDB does not use migrations, instead, the schema is defined in the model itself)

11. create the controller (Now we are ready to craete controller) 

  	rails g controller people index new create edit update destroy

12. set Routes (config/routes.rb)

    ```
  	Rails.application.routes.draw do
     resources :people, except: [:show]
      root to: "people#index"
  	end
    ```

13. Next let's add in the controller code. Open up app/controllers/people_controller.rb and modify it so that it looks like the code listed below.(app/controllers/people_controller.rb:)

    ```
  	class PeopleController < ApplicationController
  	  def index
  	    @people = Person.all
  	  end

  	  def new
  	    @person = Person.new
  	  end

  	  def create
  	    @person = Person.new(person_params)
  	    if @person.save
  	      redirect_to people_path, notice: "The person has been created!" and return
  	    end
  	    render 'new'
  	  end

  	  def edit
  	    @person = Person.find(params[:id])
  	  end

  	  def update
  	    @person = Person.find(params[:id])
  	    if @person.update_attributes(person_params)
  	      redirect_to people_path, notice: "#{first_name} #{last_name} has been updated!" and return
  	    end
  	    render 'edit'
  	  end

  	  def destroy
  	    @person = Person.find(params[:id])
  	    @person.destroy
  	    redirect_to people_path, notice: "#{first_name} #{last_name} has been deleted!" and return
  	  end

  	private

      def person_params
    	 params.require(:person).permit(:first_name, :last_name, :email, :notes)
      end
  	end
    ```

14. create the view (Now we are ready to craete view)

  	14.1 Before we get started on the views, let's add Bootstrap to the project to pretty things up a bit. Open up the app/views/layouts/application.html.erb file and modify it so that it looks like the code listed below.(app/views/layouts/application.html.erb:)


      ```
    	<!DOCTYPE html>
    	<html>
    	<head>
    	  <title>MongoDBExampleApp</title>
    	  <%= stylesheet_link_tag 'https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css' %>
    	  <%= javascript_include_tag 'https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js' %>
    	  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
    	  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
    	  <%= csrf_meta_tags %>
    	</head>
    	<body>
    	  <div class="container">
    	    <%= yield %>
    	  </div>
    	</body>
    	</html>
      ```

  	14.2 Next up, let's create a file called _form.html.erb in the app/views/people folder. This file will store our form that will be used to both create an exist an existing person. Once you create the file, add in the code listed below.(app/views/people/_form.html.erb:)

      ```
    	<%= form_for @person do |f| %>
    	  <div class="form-group">
    	    <%= f.label :first_name %>
    	    <%= f.text_field :first_name, class: "form-control" %>
    	  </div>
    	  <div class="form-group">
    	    <%= f.label :last_name %>
    	    <%= f.text_field :last_name, class: "form-control" %>
    	  </div>
    	  <div class="form-group">
    	    <%= f.label :email %>
    	    <%= f.text_field :email, class: "form-control" %>
    	  </div>
    	  <div class="form-group">
    	    <%= f.label :notes %>
    	    <%= f.text_area :notes, class: "form-control" %>
    	  </div>
    	  <%= f.submit class: "btn btn-primary" %>
    	<% end %>
      ```

	14.3 Next, let's open the new person view at app/views/people/new.html.erb and add in the code listed below. This will include the form we just created so that we can create a new person.(app/views/people/new.html.erb:)

    ```
   	 <h1>New Person</h1>
  	<div class="well">
  	  <%= render "form" %>
  	</div>
    ```

	14.4 Now, let's modify our edit person view and add in the form. Open up the edit person view and modify it so that it looks like the code listed below.(app/views/people/edit.html.erb:)

    ```
    <h1>Edit Person</h1>
    <div class="well">
      <%= render "form" %>
    </div>
    ```

	14.5 Now, let's create our index view. Open up the index view for our people controller at app/views/people/index.html.erb and modify it so that it looks like the code listed below.(app/views/people/index.html.erb:)

    ```
  	<h1>People</h1>
  	<div class="well">
  	  <%= link_to "New Person", new_person_path, class: "btn btn-primary" %>
  	</div>

  	<% if flash[:notice] %>
  	  <div class="alert alert-success"><%= flash[:notice] %></div>
  	<% end %>

  	<table class="table table-bordered table-striped">
  	  <thead>
  	    <tr>
  	      <th>First Name</th>
  	      <th>Last Name</th>
  	      <th>Email</th>
  	      <th>Notes</th>
  	      <th>&nbsp;</th>
  	      <th>&nbsp;</th>
  	    </tr>
  	  </thead>
  	  <tbody>
  	    <% if @people.size == 0 %>
  	      <tr>
  	        <td colspan="6">No people found.</td>
  	      </tr>
  	    <% else %>
  	      <% @people.each do |person| %>
  	        <tr>
  	          <td><%= person.first_name %></td>
  	          <td><%= person.last_name %></td>
  	          <td><%= person.email %></td>
  	          <td><%= person.notes %></td>
  	          <td>
  	            <%= link_to "Edit", edit_person_path(person), class: "btn btn-default" %>
  	          </td>
  	          <td>
  	            <%= button_to "Delete", person, method: :delete, data: { confirm: "Are you sure you wish to delete #{person.first_name} #{person.last_name}?" }, class: "btn btn-danger" %>
  	          </td>
  	        </tr>
  	      <% end %>
  	    <% end %>
  	  </tbody>
  	</table>
    ```

15. Great! Now if we start our Rails server using the rails scommand and navigate to our rails development site at https://localhost:3000 we will find that we have a fully simple, functioning app using MongoDB.

  ```
	rails s

	https://localhost:3000
  ```
