# Ruby On Rails

## Creating the Application
1. Create a new project with `rails new blog`
    * This creates a Rails application and installs gem dependencies already mentioned in Gemfile using `bundle install`
2. Blog directory has a number of auto-generated files and folders that make up the rails application.

|File/Folder|Purpose|
|---|---|
|app/|Contains controllers, models, views, helpers, mailers, assets etc...|
|bin/|Contains rails script that starts the application and contains other scripts used to setup, update, deploy or run the application|
|config/|Configures application routes, database etc...|
|config.ru|Rack configuration for rack servers used to start the application|
|db/|Contains current database schema and database migrations|
|Gemfile & Gemfile.lock|Allow specifications of which gem dependencies required for Rails application. Files used by Bundler gem.|
|lib/|Extends modules for your application|
|log/|Application log files|
|package.json|Specifies which npm dependenicies needed for Rails application. Used by Yarn|
|public/|Contains static files and compiled assets|
|Rakefile|Locates and loads tasks that can be run from command line. Rather than changing Rakefile, add own tasks by adding files to the `lib/tasks` directory of the application|
|test/|Unit tests, fixtures and other testing|
|tmp/|Temporary cache and pid files|
|vendor/|Place for third party code|

## Sarting the Web Server
1. Start web server on development machine
    * `bin/rails server`
    * If there are any errors, run `bundle install` to install dependencies from the Gemfile

## Creating a Basic Display
1. At minimum, create a controller and view
2. Controllers purpose is to recieve specific requests for the application
3. Routing decides which controller recieves which requests
4. There can be more than one route to each controller
5. Different routes can be served by different actions
6. Each actions purpose is to collect information to provide it to a view
7. A views purpose is to display information in human readable format
8. The controller collects the information, but the view just displays it
9. View templates are written in eRuby (embedded ruby) processed by request cycle in Rails before being sent to the user
10. To create a new controller called "welcome" with an action called "index" run:
    * `bin/rails generate controller Welcome index`
    * Rails will create several files and routes

## Setting Applications Home Page
1. Need to configure routing so that upon navigating to root URL, the welcome view is displayed
2. In `config/routes.rb`
    * This is applications routing file that holds entries in DSL (domain specific language) telling rails how to connect incoming requests to controllers and actions.
    ```
    Rails.application.routes.draw do
        get 'welcome/index'
        root 'welcome#index'
    end
    ```
    * root `welcome#index` tells Rails to map requests to the root of the applicaton to the welcome controller's index action and `get 'welcome/index'` tells Rails to map requests to `http://localhost:3000/welcome/index`

## Creating Resources
* Resources are used as a collecton of similar objects. Can CRUD items FOR a resource.
* Rails has a resource method used to declare a standard REST resource.
    * Add the resource to the routes `resources :articles`
    * Running `bin/rails routes` standard RESTful actions will have already defined routes

|Prefix|Verb|URI Pattern|Controller#Action
|-----------------|-----------|-----------|----|
|welcome_index|GET|/welcome/index(.:format)|welcome#index|
|articles|GET|/articles(.:format)|articles#index|
|articles|POST|/articles/(.:format)|articles#create|
|new_article|GET|/articles/new(.:format)|articles#new|
|edit_article|GET|/articles/:id/edit(.:format)|articles#edit|
|article|GET|/articles/:id(.:format)|articles#show|
|article|PATCH|/articles/:id(.:format)|articles#update|
|article|PUT|/articles/:id(.:format)|articles#update|
|article|DELETE|/articles/:id(.:format)|articles#destroy|
|.root|GET|/|welcome#index|

## Creating Controllers and Views
* Create the page that will display the form for creating a new article
* With the resource already defined in the route, requests can now be made to all the above paths (such as /articles/new)
* Create a controller called `ArticlesController` with command `bin/rails generate controller Articles`
* A controller is a class defined to inherit from ApplicationController. Inside this class, define methods that becomes actions for the controller. Actions perform CRUD operations on articles in the system.
    * There are `public`, `private`, and `protected` methods, but only __public__ methods can be actions for controllers
* Define a new action inside a controller - in this case, we define a new action called "new" because that is the page we want users to navigate to to create a new page.
```articles_controller.rb
class ArticlesController < ApplicationController
    def new
    end
end
```
* Rails expects plain actions to have views associated with them to display their information. A simple template to be rendered would be located at `app/views/articles/new.html.erb`. The first extension is the __format__, followed by the __handler__ used to render the template. 
    * Rails attempts to find a template called articles/new within app/views for the application. The format can __only__ be html and the default is __erb__.
    * Rails can use handlers like __builder__ for XML and __coffee__ for CoffeeScript
    * Create a new file at `app/views/articles/new.html.erb`

## The First Form
1. Use a form builder with form_with.
2. Add code into `app/views/articles/new.html.erb`
```
<%= form_with scope: :article, local: true do |form| %>
  <p>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </p>
 
  <p>
    <%= form.label :text %><br>
    <%= form.text_area :text %>
  </p>
 
  <p>
    <%= form.submit %>
  </p>
<% end %>
```

* `form_with` passes an identifying scope for the form - in this case it is __:article__. This tells the form_with helper what this form is for.
* Inside the block for this method, the __FormBuilder__ object represented by __form__ is used to build two labels and two text fields, as well as a submit button.
* Must add the appropriate action to the form upon submitting
* Modify the form_with and add the url to look like this:
```
<%= form_with scope: :article, url: articles_path, local: true do |form| %>
```
* A __articles_path__ helper is passed to the :url option.
* The articles_path tells Rails to point to the form in the URI pattern associated with the articles prefix, and the form will send (by default) a POST request to that route.
* This is associated with the create action of the current controller.
* Upon submit, since there is no action in the controller to handle this request, there will be an error.

## Creating Articles
1. Create a new action in the controller called __create__.
2. Resubmitting now, we get another error because we don't specify the response. The create action should save the article to the database.
3. Upon submission of a form, fields of the form are sent to Rails as paramters. These can be referenced inside the controller actions to perform a particular task.
    * To see the parameters, use the code below:
    ```
    def create
        render plain: params[:article].inspect
    end
    ```
4. The render method takes a hash with a key of __:plain__ and value of __params[:article].inspect__. The params method is the object that represents the parameters coming from the form.
    * The params method returns an __ActionController::Parameters__ object that allows access to keys of the hash using strings or symbols.
    * So on submit now, it would return
    ```
    <ActionController::Parameters {"title"=>"Jessica", "text"=>"Wong"} permitted: false>
    ```

## Creating a model
1. Models in Rails uses a singular name, while corresponding databases use a plural name. Rails has a generator for creating models in terminal
    * `bin/rails generate model Article title:string text:text`
    * Tells Rails we want an Article model with a title attribute of type string and text attribute of type text.
    * Those attributes are auto added to the articles table in the database and mapped to the Article model

## Running Migrations
1. `bin/rails generate model` creates a database migration file inside the db/migrate directory.
    * Migrations are Ruby classes designed to simply create and modify database tables.
    * Uses rake commands to run migration, and undo them after they've been appplied.
2. To make the migration, run `bin/rails db:migrate`
    * This is in the development environment (config/database.yml file). To execute migrations in another environment, explicitly pass it when invoking the command: `bin/rails db:migrate RAILS_ENV=production`

## Saving data in the controller
1. Change the create action to use the Article model and save data in the database.
```
def create
    @article = Article.new(params[:article])

    @article.save
    redirect_to @article
end
```
2. Every Rails model can be initialized with its respective attributes which are automatically mapped to the respective database columns - done in the first line.
    * The __@article.save__ saves the model in the database and returns a boolean, and then the user is redirected to the `show action`
3. Whitelist the parameters being saved into the database
    * This case, allow and require the __text__ and __title__ parameters for valid use of create. The syntax introduces require and permit.
    ```
    def create
        @article = Article.new(article_params)

        @article.save
        redirect_to @article
    end

    private
        def article_params
            params.require(:article).permit(:title, :text)
        end
    ```

## Showing one Article
1. It expects an :id parameter, and must have the action in the controller. Make sure this goes at the top.
```
  def show
    @article = Article.find(params[:id])
  end
```
2. Use Article.find to search for article of interest, and the params[:id] to get the :id parameter from the request.
3. Use an instance variable to hold a reference to the article object.
    * Rails will pass all instance variables to the view and can be accessed with dot notation
    ```
    <p>
    <strong>Title:</strong>
    <%= @article.title %>
    </p>
    
    <p>
    <strong>Text:</strong>
    <%= @article.text %>
    </p>
    ```

## Showing all Articles
1. Add the index action inside ArticlesController.
```
def index
    @articles = Article.all
end
```
2. Add a view for this action in path `app/views/articles/index.html/erb`
```
<h1>Listing articles</h1>
 
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
    <th></th>
  </tr>
 
  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
    </tr>
  <% end %>
</table>
```
* Using a for loop, print each article with a link to show details of the id.

## Adding Links
1. At the index page, add a link with the following line of code
    * `<%= link_to 'My Blog', controller: 'articles' %>`
    * The __link_to__ method is a Rails helper and creates a hyperlink based on text to display and where to go.
    * Since this is the index, the action does not need to be specified. 
2. In the index page of the articles, add a "New Article" link
    * `<%= link_to 'New Article', new_article_path %>`
3. In the new article page, add a "Back" button to go back to the list of articles
    * `<%= link_to 'Back', article_path %>` or `<%= link_to 'Back',controller: articles %>`
4. If you want to link to an action in the same controller, don't need to specify the __:controller__ option as Rails uses the current one by default.

## Adding Validation
1. Rails includes methods to valudate data sent to the models. Add the following to the `app/models/article.rb`
    * `validates :title, presence: true, length: {minimum: 5}`
    * This ensures all articles have a title that is at least 5 characters long. Rails validates a variety of conditions including the presence or uniqueness of columns, their format, and existence of associated objects.
    * Now, if `@article.save` is called on an invalid article, it will return __false__. Modify the controller to render the form again if saving fails.
    * We render the 'new' action again which creates a new instance of the article
    ```
    def new
        @article = Article.new
    end

    def create
        @article = Article.new(article_params)

        if @article.save
            redirect_to @article
        else
            render 'new'
        end
    end
    ```
2. In 
    * We check for errors with `@article.errors.any?` and then show a list of all errors with `@article.errors.full_messages`
    * __pluralize__ is a rails helper that takes a number and string as arguments. If the number is greater than one, the string is automatically pluralized.
    * Addition of `@article = Article.new` in the ArticlesController is because @article would be nil otherwise.
    * Rails automatically wraps fields that contain an error with a div class `field_with_errors`

## Updating Articles
1. Add an edit action to the ArticlesController between the new and create actions
```
def edit
    @article = Article.find(params[:id])
end
```
2. The view contains a form similar to the one used when creating new articles.
3. In the edit view, add the code:
```

<h1>Edit article</h1>
 
<%= form_with(model: @article, local: true) do |form| %>
 
  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %> prohibited
        this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
 
  <p>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </p>
 
  <p>
    <%= form.label :text %><br>
    <%= form.text_area :text %>
  </p>
 
  <p>
    <%= form.submit %>
  </p>
 
<% end %>
 
<%= link_to 'Back', articles_path %>
```
* This form points to the update action
* Submitting the form passes the article object to the method which automatically creates the url for submitting the edited article.
* Rails submits the form via the PATCH HTTP method which is the method expected to update resources according to the REST protocol.
* Arguments of __form_with__ could be model objects such as __model: @article__ which cause the helper to fill in the form with fields of the object.
* Passing in a symbol scope __scope: :article__ creates the fields but without anything filling them (like in new).
4. Create the update action in the controller
```
def update
    @article = Article.find(params[:id])

    if @article.update(article_params)
        redirect_to @article
    else
        render 'edit'
    end
end
```
* Update used to update an existing record, and accepts a hash containing attributes to update.
* Not all attributes need to be passed
* Add a link to the edit page on the index page `<td><%= link_to 'Edit', edit_article_path(article) %></td>`

## Using Partials
1. Edit page is similar to new page in terms of code. Remove duplication by using partial views. Partial views are prefixed with an underscore.
2. Create a new partial view called `_form.html.erb` with code
```
<%= form_with model: @article, local: true do |form| %>
 
  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %> prohibited
        this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
 
  <p>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </p>
 
  <p>
    <%= form.label :text %><br>
    <%= form.text_area :text %>
  </p>
 
  <p>
    <%= form.submit %>
  </p>
 
<% end %>
```
* Everything except the __form_with__ declaration remained the same, but this can be used to stand in for either because __@article__ is a resource corresponding to a full set of RESTful routes, which Rails can infer which URI method to use.
3. Update the __new.html.erb__ and __edit.html.erb__ view to use the partial view.

## Deleting Articles
1. Delete routing method used for routes that destroy resources. This route is mapped to the destroy action inside the constroller. Destroy is the last CRUD action in the controller but must be placed before any private or protected method.
```
def destroy
    @article = Article.find(params[:id])
    @article.destroy

    redirect_to articles_path
end
```
2. Can call destroy on active record objects when want to delete them from database. No need to add view for this action because we are redirecting.
3. Add destroy link to index action template.
```
<td><%= link_to 'Delete', article_path(article), method: :delete, data: {confirm: 'Are you sure?'} %></td>
```
* We pass the named route as the second argument and then options as another argument.
* The __method: :delete__ and __data: {confirm: 'Are you sure?'}__ options are used as HTML5 attributes. When link is clicked, Rails shows a confirm dialog to the user then submits the link with method delete.
* This is done with JavaScript file __rails-ujs__ which is automatically included in applications layout.

## Adding a Second Model
1. Create a comment model that references one article. In terminal, run command `bin/rails generate model Comment commenter:string body:text article:references`
2. This generates four files

|File|Purpose
|----|-------|
|db/migrate/xxxxxx_create_comments.rb|Migration to create the comments table in the database|
|app/models/comment.rb|The comment model|
|test/models/comment_test.rb|Testing harness for the comment model|
|test/fixtures/comments.yml|Sample comments for use in testing|

3. In the `app/models/comment.rb`, the line __belongs_to :article__ sets an Active Record association.
    * The __(:refereces)__ keyword is a special data type for models by creating a new column in the database table with the provided model name appended with an _id that can hold integer values.
    * rails also makes a migration to create corresponding database table that includes t.references that creates an integer column called __article_id__, an index, and foreign key constraint that points to the id column in the articles table.

## Associating Models
1. Active record associations allow declaration of relationships between two models.
    * Each comment belongs to one article
    * One article has many comments
2. Association created automtically inside the Comment model that makes each comment belong to one Article.
3. Edit `app/models/article.rb` to add the other side of the association with `has_many :comments`
4. The declarations enable automatic behaviour. Ex. with instance variable @article containing an article, you can retrieve all comments belonging to that article as an array using `@article.comments`.

## Adding route for comments
1. Add routes so Rails knows where to navigate for comments.
```
resources :articles do
  resources :comments
end
```
* Creates comments as a nested resource within articles.

## Generating a Controller
1. Create a matching controller with the command `bin/rails generate controller Comments`
2. This creates 5 files and one empty directory

|File/Directory|Purpose|
|--------------|-------|
|app/controllers/comments_controller.rb|The comments controller|
|app/views/comments/|Views of the controller stored here|
|test/controllers/comments_controller_text_rb|Test for the controller|
|app/helpers/comments_helper.rb|A view helper file|
|app/assets/javascripts/comments.coffee|CoffeeScript for the controller|
|app/assets/stylesheets/comments.scss|CSS for controller|

3. Alter show.html.erb to allow users to write a comment
```
<h2>Add a comment:</h2>
<%= form_with(model: [ @article, @article.comments.build ], local: true) do |form| %>
  <p>
    <%= form.label :commenter %><br>
    <%= form.text_field :commenter %>
  </p>
  <p>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </p>
  <p>
    <%= form.submit %>
  </p>
<% end %>
```
* Adds a form to article show page that creates a new comment by calling the CommentsController create action.
* form_with call uses an array which builds a nested route. Ex __/articles/1/comments__
* Add the create section in the `app/controllers/comments_controller.rb`
```
    def create
        @article = Article.find(params[:article_id])
        @comment = @article.comments.create(comment_params)
        redirect_to article_path(@article)
    end

    private
    def comment_params
        params.require(:comment).permit(:commenter, :body)
    end
```
* Each request for a comment has to keep track of the article to which the comment is attached
* First, use find method of Article model to get article in question
* Use methods available for associations. Use create method on @article.comments to create and save the comment. This auto links the comment to belong to that particular article.
* Once new comment is made, send user back to original article using article_path(@article) helper - this renders the show.html.erb template. Add code to view
```
<h2>Comments</h2>
<% @article.comments.each do |comment| %>
<p>
    <strong>Commenter:</strong>
    <%= comment.commenter %>
</p>
<p>
    <strong>Comment</strong>
    <%= comment.body%>
</p>
<%end%>
```

## Refactoring
* Render partials collections for __show.html.erb__
    1. Make comment partials to extract showing all comments for the article.
    ```
    <p>
    <strong>Commenter:</strong>
    <%= comment.commenter %>
    </p>

    <p>
        <strong>Comment:</strong>
        <%= comment.body %>
    </p>
    ```
    2. Modify the __show.html.erb__ file to include the partials
    ```
    <h2>Comments</h2>
    <%= render @article.comments %>
    ```
    * This renders the partial view once for each comment in the @article.comments collection. It assigns each comment to a local variable named the same as the partial

## Rendering a partial form
1. Move the new comment section into its own partial form
2. Alter the __show.html.erb__ to include the partial
    ```
    <h2>Add Comments</h2>
    <%= render 'comments/form' %>
    ```
    * This render defines the partial template to render __comments/form__. Rails spots the forward slash in string and knows to render _form.html.erb in the app/views/comments directory
    * __@article__ object is available to any partials rendered in the view because it was defined as an instance variable.

## Deleting Comments
1. Implement a link in the view and a destroy action in the CommentsController
```
<%= link_to 'Remove Comment', [comment.article, comment], method: :delete, data: {confirm: 'Are you sure?'} %>
```
* This link fires a "delete" method to the CommentsController which finds the comment to delete. It uses the url __/articles/:article_id/comments/:id__. The two params are given in the second option. 
2. In the controller, handle it.
```
def destroy
    @article = Article.find(params[:article_id])
    @comment = @article.comments.find(params[:id])
    @comment.destroy
    redirect_to article_path(@article)
end
```
* The destroy action finds the article and locates the comment within the __@article.comments__ collection and removes it from the database and sends user back to show action for the article.

## Deleting Associated Objects
1. If an article is deleted, associated comments must also be deleted. Use the dependent option of association to achieve this.
2. Modify __app/models/article.rb__ to `has_many :comments, depdendent: :destroy`

## Security
### Basic Authorization
1. Rails has HTTP authentication system.
2. In ArticlesController, block access to various actions if person is not authenticated. Use the `http_basic_authenticate_with` method to allow access to requested action if that method allows it. 
3. To use the authentication system, specify at top of ArticlesController in __app/controllers/articles_controller.rb__. Want use authentication on every action except index and show.
```articles_controller.rb
http_basic_authenticate_with name: "dhh", password: "secret", except: [:index, :show]
```
```comments_controller.rb
http_basic_authenticate_with name: "dhh", password: "secret", only: :destroy
```
* Note the difference between authentications are __except__ and __only__. Where except excludes those actions, and only includes those actions.


## Tips and Tricks
* Use square brackets to access values of properties from classes
* Use the __@__ symbol to define a new instance variable
* Use <%= %> to enclode ruby code in html
* To add a new model:
    1. `bin/rails generate model (name_of_model) (column_name):(data_type) [(column_name):(data_type)...]`
        * Data types include: __string__, __text__, __references__
* To add a controller:
    1. `bin/rails generate controller (name_of_controller)`