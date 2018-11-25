---
layout: post
title: "Deploying your Sinatra application"
type: code
date: 2014-12-10 18:07 -0500
---
_Originally posted on [Medium](https://medium.com/@pam_yam/deploying-your-sinatra-application-bf8b0c4b83f8)._

## A beginner’s guide

I cannot lie. My first exposure to Sinatra was rather quick and incomplete. I didn’t explore as much as I should have, and the minute I got the chance to switch to Rails, I forgot all about Sinatra because Rails does a lot for you. I was being a lazy programmer but not in a pedagogical way.

Lately however, I’ve had to revisit Sinatra, make apps with it, and, as of a few days ago, deploy with it. I’m seeing how awesome Sinatra truly is, which is pretty cool! Better late than never, am I right?

In this tutorial we will be making a Bucket List with Sinatra, Postgresql , and ActiveRecord, then we will deploy it to Heroku. It is a simple CRUD application that keeps both our database and logic on the small side.

#### First things first:
**Starting your app with git**
Let’s start by making a bucket folder from our terminal.

{% highlight bash %}
$ mkdir bucket
{% endhighlight %}

Within bucket, we want to start a git repository. We will then create one on GitHub and add it as a remote. When our repo is initiated, we will add all the folders necessary to start building our application.

{% highlight bash %}
$ cd bucket
$ git init
$ mkdir public views models
$ touch app.rb
$ touch readme.md
$ git add .
$ git commit -m 'initial commit'
$ git remote add origin git@github.com:YOUR_USERNAME/YOUR_REPO_NAME
{% endhighlight %}

#### Adding your dependencies:

**Making a Gemfile**
Once our repo has been set up, we can go ahead and add a Gemfile. But first, let’s talk about what the Gemfile does for us. Let’s say Joanna wants to run your application on her computer but she doesn’t have the same environment as yours. In her terminal and in the application’s directory, she can just run bundle install and voila! Her environment is now ready to work with your application. Now, let’s say that Heroku is a more powerful version of Joanna. In order to be able to set up the proper environment for your application on the server side, Heroku has to install all the necessary gems. Hence the Gemfile.

In your Gemfile, add your gems:

{% highlight ruby %}
# in your Gemfile
source 'https://rubygems.org'
#your version of ruby
ruby '2.1.2'
gem 'sinatra'
gem 'activerecord'
gem 'sinatra-activerecord'
gem 'pg'
{% endhighlight %}

Run `bundle install` in your terminal. If it wasn’t before, your environment is ready for this application! Now is a good time to commit those changes to your repo.

#### Building your database with rake:
**Rakefile and migrations**

Create a config folder, where you will set up your database and your environments. From the command line and within your bucket folder:

{% highlight bash %}
$ mkdir config
$ touch config/database.yml
$ touch config/environments.rb
{% endhighlight %}

`database.yml` will hold all the info necessary to connect to your local database, and `environments.rb` will actually connect to your database through `ActiveRecord` in both development and production environments.

{% highlight yml %}
#in database.yml
development:
  adapter: postgresql
  database: bucket_dev
  host: localhost
{% endhighlight %}

{% highlight ruby %}
# in environments.rb
configure :production, :development do
db = URI.parse(ENV['DATABASE_URL'] || 'postgres://localhost/bucket_dev')
ActiveRecord::Base.establish_connection(
  adapter: db.scheme == 'postgres' ? 'postgresql' : db.scheme,
  host: db.host,
  username: db.user,
  password: db.password,
  database: db.path[1..-1],
  encoding: 'utf8'
)
end
{% endhighlight %}

Now that our database is setup, let’s create a Rakefile from the command line:

{% highlight bash %}
$ touch Rakefile
{% endhighlight %}

In our case, the Rakefile will make it possible to create a database and a schema in Ruby by running rake tasks. We simply need it to be aware of our application and the rake gem. Let’s add all this information:

{% highlight ruby %}
#in the Rakefile
require './app'
require 'sinatra/activerecord/rake'
Almost ready to make our database. We need to add two lines to our app.rb first:

#in app.rb
require 'sinatra'
require 'sinatra/activerecord'
{% endhighlight %}

Now, we can create our database, and add the tables we need for our full fledge application. Since we’re making a bucket list, we will just require a table for our items. Let’s do it:

{% highlight bash %}
$ rake db:create
$ rake db:create_migration NAME=create_items
{% endhighlight %}

The latter command will generate a file entitled [`TIMESTAMP]_create_items.rb` within a `migrate` folder, itself nested inside a `db` folder. This should be our new folder’s path:

`db/migrate/[TIMESTAMP]_create_items.rb`

Just like that, we created what will be a future table in our database. Before both can be connected, we will first specify the columns we want within our table:

{% highlight ruby %}
#in [TIMESTAMP]_create_items.rb
class CreateItems < ActiveRecord::Migration
  def change
    create_table :items do |t|
      t.string :title, null: false
      t.text :desc, default: ""
      t.boolean :is_done, default: false
      t.timestamps
    end
  end
end
{% endhighlight %}
Now, to add this table and its columns to our database, we simply need to run the command that will generate our schema.

{% highlight bash %}
$ rake db:migrate
{% endhighlight %}

Okay, let’s review. Using rake tasks, we were able to create our database, then create a migration. Migrations are an Active Record feature that allows us to alter our database’s schema using Ruby instead of pure SQL (Check out the Active Record docs for more details on the topic). We then added some columns to our items table:

- A column for an item’s title as a string
- A column for an item’s description as text (more characters allowed)
- A column for an item’s completed status (‘is_done’) as a boolean
- A column for our items’ timestamps

There. Our database work is complete. We can now move on to making the actual application. Let’s add our item model to our models folder from our command line:

{% highlight bash %}
$ touch models/item.rb
{% endhighlight %}

Now, let’s configure our model to ensure that each new item will at least have a title:

{% highlight ruby %}
#in item.rb
class Item < ActiveRecord::Base
  validates_presence_of :title
end
{% endhighlight %}

That’s it for our model. Let’s move on to setting up our routes in app.rb.

{% highlight ruby %}
#in app.rb
require 'sinatra'
require 'sinatra/activerecord'
require_relative './models/item'
require_relative './config/environments'

#setting up the index route
get '/' do
  @items = Item.all
  erb :index
end

#setting up the form view
get '/new' do
  erb :new_form
end

#setting up the post route for our form
post '/new' do
  @item = Item.new(title: params[:title], desc: params[:desc])
  if @item.save
    redirect '/'
  else
    @errors = @item.errors.full_messages
    render '/new'
  end
end
{% endhighlight %}

Let’s add some views for our application. From the command line:

{% highlight ruby %}
$ touch public/layout.erb
$ touch public/index.erb
$ touch public/new_form.erb
{% endhighlight %}

Let’s add structure to those views:

{% highlight erb %}
<!-- in layout.erb -->
<!DOCTYPE html>
<html>
  <head>
    <title></title>
    <link type="text/css" rel="stylesheet" href="/styles.css">
  </head>
  <body>
    <%= yield %>
  </body>
</html>
{% endhighlight %}

{% highlight erb %}
<!-- in index.erb -->
<% if @items.empty? %>
  <h2> No items in that bucket...yet! </h2>
<% else %>
  <ul>
   <% @items.each do |item| %>
    <li><%= item.title %></li>
  </ul>
  <% end %>
<% end %>
<%= erb :new_form %>
{% endhighlight %}

{% highlight erb %}
<!-- in new_form.erb -->
<% unless !@errors || @errors.empty %>
 <ul>
   <% @errors.each do |error| %>
     <li><%= error %></li>
   <% end %>
 </ul>
<% end %>
<form action="/new" method="post">
  <label>Title</label>
  <input type="text" name="title" placeholder="Climb every mountain">
  <label>Description </label>
  <textarea name="desc"></textarea>
  <input type="checkbox" value="checked" name="is_done" %>
  <input type="submit">
</form>
{% endhighlight %}

Commit those changes and let’s add some simple styling. Add a `styles.css` file to your `public` folder. Add the following lines in there, and more if you’re feeling the CSS vibes.

{% highlight css %}
//in styles.css
body {
 text-align: center;
 width: 100%;
 font-family: "Avenir";
}
ul{
 list-style: none;
 padding: 0;
}
a{
 color: inherit;
 text-decoration: none;
}
input,
label,
textarea{
 margin: 18px auto;
 display: block;
}
small{
 font-weight: lighter;
}
{% endhighlight %}

Commit those changes and let’s check out our app. From your terminal, you will start your server by running your application like so:

{% highlight bash %}
$ ruby app.rb
{% endhighlight %}

In your browser, visit `http://localhost:4567`. That's your application!

You’re now ready to deploy.

#### Deploying to Heroku:

**Final stretch**
You will have to create one more file before deploying: `config.ru`. It will hold all the information that Heroku needs to run your application.
At the root of your application, create the file from your terminal.

{% highlight bash %}
$ touch config.ru
{% endhighlight %}

Then add those lines to it:

{% highlight ruby %}
require './app'
run Sinatra::Application
{% endhighlight %}

This will let Heroku know that this is a Sinatra application.
Now commit your changes, and push them to your GitHub repo.

{% highlight bash %}
$ git add .
$ git commit -m 'ready for deployment'
$ git push origin master
{% endhighlight %}

Now you can deploy! Assuming that you already have an account on Heroku, run those commands in your terminal:

{% highlight bash %}
$ heroku create
$ git push heroku master
{% endhighlight %}

Almost done! Now let’s set up our database on Heroku:

{% highlight bash %}
$ heroku run rake db:migrate
{% endhighlight %}

And done!! You should now be able to open your application:

{% highlight bash %}
$ heroku open
{% endhighlight %}

Your application should be live on the interweb now! Go ahead and share it with your friends!

_Originally posted on [Medium](https://medium.com/@pam_yam/deploying-your-sinatra-application-bf8b0c4b83f8)._