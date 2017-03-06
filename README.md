#Rock 'n Rails!

<!-- BEGIN SF-WDI-LABS BADGES -->
<!-- INSTRUCTOR TODO: Make sure to manually bump version number of commits-since ("updates") badge to latest release version -->
[![Current Labs Version](https://img.shields.io/github/tag/SF-WDI-LABS/rock-n-rails.svg?label=sf-wdi-labs)](https://github.com/SF-WDI-LABS/rock-n-rails)
[![Issues Count](https://img.shields.io/github/issues-raw/SF-WDI-LABS/rock-n-rails.svg)](https://github.com/SF-WDI-LABS/rock-n-rails/issues)
<!-- END SF-WDI-LABS BADGES -->

For this morning exercise we're going to be synthesizing all our Rails knowledge to build a album collection! At the bottom of this file you can find a link to a completed solution.

###User stories

*User should be able to...*

1. See all the albums on `albums#index`
    - with an album `title` and `artist`
    - and be able to click on an individual album and be redirected to the show page
    - and see a link to `Create New Album`
2. See a single album on `album#show`
    - with an album `title`, `artist`, `year` and `cover_art`
3. See a form to create a new album on `album#new`
4. Submit the new album form to `album#create` to create a new album and then be redirected back to album index.

## Initial Setup
To create a new rails application, from the command line run:

```
rails new rock-n-rails-demo-app --database=postgresql --skip-test --skip-bundle
```

Rails uses coffee-script (`.coffee`) for `.js` and sass (`.scss`) for `.css`.

Let's keep it simple. Please remove both dependencies from your `Gemfile`:

```
gem 'sass-rails'    # remove this line
gem 'coffee-rails'  # remove this line
```

Make sure you've saved it, and then run:

```bash
bundle install
# or
bundle
```

Now you should be able to launch your local server:

```bash
rails server
# or
rails s
```

Now open your browser to `localhost:3000`.


(STOP and COMMIT!) -- don't forget to `git init`

## Model
Let's take a moment to create and seed our database.

#### `Album` Model
A `Album` should have the following attributes:

* title — String
* artist — String
* year — Integer
* cover_art — String
* song_count — Integer

```bash
rails g model album title:string artist:string year:integer cover_art:string song_count:integer
```

* Create a database for your application to use

```bash
rails db:create
```

* Run the migration that was generated to create a new table in the database.

```bash
rails db:migrate
```

* Play with your new `Album` model in the rails console:

```bash
rails console
> Album.all #=> []
> Album.create({title: "Test Album"})
```

> Note that ActiveRecord displays the actual SQL query in your console. Make sure to read these queries -- it's a great way to practice!

(STOP and COMMIT)

#### `Album` seed task
* In `db/seeds.rb` create some albums!

`db/seeds.rb`.

```ruby
# Wipe the database
Album.destroy_all
# Let's create a bunch of albums
Album.create([
  {
    title: "On Avery Island",
    artist: "Neutral Milk Hotel",
    year: 1996,
    cover_art: "https://upload.wikimedia.org/wikipedia/en/7/73/On_avery_island_album_cover.jpg",
    song_count: 12
  },
  {
    title: "Everything All the Time",
    artist: "Band of Horses",
    year: 2006,
    cover_art: "https://upload.wikimedia.org/wikipedia/en/5/51/BandofHorsesEverythingalltheTime.jpg",
    song_count: 10
  },
  {
    title: "The Flying Club Cup",
    artist: "Beirut",
    year: 2007,
    cover_art: "https://upload.wikimedia.org/wikipedia/en/4/4c/The_Flying_Club_Cup.jpg",
    song_count: 13
  }
])
```

* Run the seed file!

```bash
rails db:seed
```

* Check that everything was done correctly, run `rails console` or just `rails c` and inside run `Album.all`. Make sure that you can see an array of all the albums from your seed file. Exit by typing `exit`.

(STOP and COMMIT)


## View, Routes, and Controllers
**See all the albums on `albums#index`**

* Visit `localhost:3000/albums` in your browser
    - You should see an error complaining that "no route matches...". What does that tell you?

* Let's add our first RESTful route for our `Albums` resource!

In `config/routes.rb`, add the following route(s):

```ruby
get "/albums" => "albums#index", as: 'albums'  # add me!
#get "/albums/new" => "albums#new", as: 'new_album'
#get "/albums/:id" => "albums#show", as: 'album'
#post "/albums" => "albums#create"
```

* Now, refresh the page, and you should see it complain about a missing controller!

Let's generate our albums controller!

```bash
rails g controller albums
```

In `albums_controller.rb` let's add:

```ruby
def index
  render :index # optional
end
```

* Refresh the page, and you should see it complain about a missing view!

Let's create `views/albums/index.html.erb` and add the following html:

```html
<h1>Rock 'n Rails! (albums#index)</h1>
```

* Refresh the page, and you should see the above HTML rendered. Yay!

(STOP and COMMIT)

Now let's connect our model. Update your `index` action in `albums_controller.rb` to grab all the albums:

``` ruby
def index
  @albums = Album.all
  # render :index
end
```

And then let's also update the view to render a list of albums:

``` html
<% @albums.each do |album| %>
  <p>Title: <%= album.title %></p>
  <p>Artist: <%= album.artist %></p>
  <img src="<%= album.cover_art %>"> <!-- TODO: smelly -->
<% end %>
```

* Refresh the page and you should see your list of albums! Good work!

(STOP and COMMIT)

**See a single album on `albums#show`**

* For each album in the `album#index` view let's create an anchor tag that will link to e.g. `albums/1`, `albums/2`, `albums/3`

`views/albums/index.html.erb`.

```html
<h1>Rock 'n Rails!</h1>
<% @albums.each do |album| %>
  <p>Title: <%= album.title %></p>
  <p>Artist: <%= album.artist %></p>
  <img src="<%= album.cover_art %>">
  <br>
  <!-- anchor tag that links to a show page -->
  <a href="/albums/<%= album.id %>">Show page</a> <!-- TODO: smelly -->
<% end %>
```

* Now click on one of the links. They don't work yet. What error do you see?

* Let's add our second RESTful route for our `Albums` resource!

In `config/routes.rb`, add the following route(s):

```ruby
get "/albums" => "albums#index", as: 'albums'
#get "/albums/new" => "albums#new", as: 'new_album'
get "/albums/:id" => "albums#show", as: 'album' # add me!
#post "/albums" => "albums#create"
```

* Refresh the page. What error do you see?

* We need to create the `albums#show` action now. And we need to grab the `id` from the parameters and use it to find the matching album in the database and pass it to the view.

`albums_controller.rb`

```ruby
  # ...

  def show
    @album = Album.find(params[:id])
    render :show #optional
  end
```

* Refresh the page. What error do you see?

* Let's create `views/albums/show.html.erb` and add the following html:

```html
<h1>Rock 'n Rails! (albums#show)</h1>
```

* Refresh the page and make sure you see the HTML above rendered.

* Now, in your `albums#show` view, `views/albums/show.html.erb` display the album that is being passed in.

```html
<img src="<%= @album.cover_art %>">
<h1><%= @album.title %></h1>
<h2>by <%= @album.artist %></h2>
<p>Year: <%= @album.year %></p>
<p>Song Count: <%= @album.song_count %></p>
<%= link_to "Back", albums_path %>
```

(STOP and COMMIT)

**See a form to create a new album on `album#new`**

* Let's create a link on *every* page that will get us to a form that creates a new album, which lives on `/albums/new`. We can edit the `application.html.erb` file which lives in `views/layouts/` to accomplish this. Inside the file add an anchor tag just above the `yield` statement in the `<body>`.

```html
<body>

<!--Every page will have this link to create a new album-->
<a href="/albums/new">Make a New Album</a>  <!-- TODO: the rails way -->

<%= yield %>

</body>
```

When you visit `localhost:3000/albums/new`, you should see an error.

* Let's add our third RESTful route for our `Albums` resource!

In `config/routes.rb`, add the following route(s):

```ruby
get "/albums" => "albums#index", as: 'albums'
get "/albums/new" => "albums#new", as: 'new_album'  # add me! order matters!
get "/albums/:id" => "albums#show", as: 'album'
#post "/albums" => "albums#create"
```

* Refresh and you should see a new error, complaining about the controller.

* We need to create the `albums#new` action now.

`albums_controller.rb`

```ruby
  # ...

  def new
    render :new #optional
  end
```

* Now we need to create `views/albums/new.html.erb` using a rails HTML form helper. Let's make all fields required.

```html
<%= form_for @album do |f| %>
  <span>Title: </span>
  <%= f.text_field :title, required: true %><br>
  <span>Artist: </span>
  <%= f.text_field :artist, required: true %><br>
  <span>Year: </span>
  <%= f.number_field :year, required: true %><br>
  <span>Cover art: </span>
  <%= f.url_field :cover_art, required: true %><br>
  <span>Song count: </span>
  <%= f.number_field :song_count, required: true %><br>
  <%= f.submit %>
<% end %>
```

* This form will not work yet. That's because we reference `@album` in the form but it's not defined. Let's define `@album` in our controller and pass it into our view. All we need it to be equal to is a new instance of a the `Album` model.

`app/controllers/albums_controller.rb`

```ruby
  # ...

  def new
    @album = Album.new
    render :new #optional
  end
```

* Refresh and you should see the rendered form!

(STOP and COMMIT)

**Submit the new album form to `album#create` to create a new album and then be redirected back to album index.**

* Now that our forms works, it will automatically `POST` to `/albums`. Try it and you'll see our next error!

* Let's add our fourth RESTful route for our `Albums` resource!

In `config/routes.rb`, add the following route(s):

```ruby
get "/albums" => "albums#index", as: 'albums'
get "/albums/new" => "albums#new", as: 'new_album'
get "/albums/:id" => "albums#show", as: 'album'
post "/albums" => "albums#create"  # add me!
```

* Nothing is happening in the `albums#create` controller as of yet so we need to actually create a new album there. In order to do that we must pull out the data submitted from our form from the `params` object and create a new album with it.

`app/controllers/albums_controller.rb`.

```ruby
  def create
    Album.create(
      # this is known as strong parameters, and is done for security purposes
      params.require(:album).permit(:title, :artist, :year, :cover_art, :song_count)
    )
  end
```

* You may wonder what all the business is with `.require(:album).permit(...)` is. This is known as [**strong parameters**](http://edgeguides.rubyonrails.org/action_controller_overview.html#strong-parameters) and tells our applications these are the fields we will accept. Its good security practice to help prevent users accidentally updating sensitive model attributes.

* Additionally we can refactor this code to make it look better. We can **encapsulate** our strong parameter logic into a method called `album_params`. Let's make that a private method, since only the controller itself will ever use it. At the bottom of `AlbumsController` we can write:

`app/controllers/albums_controller.rb`.

```ruby
# public methods up here

  private

  def album_params
    params.require(:album).permit(:title, :artist, :year, :cover_art, :song_count)
  end

end # end of class
```

* Now our `create` method can take advantage of the `album_params` method, which simply will output an object of key value pairs our `Album` model can use to create a new album. Also let's tell it to redirect to the index page once it's created the album.

`app/controllers/albums_controller.rb`.

```ruby
  def create
    Album.create(album_params)
    redirect_to('/albums')
  end
```

(STOP and COMMIT)

Congrats! We've complete all the user stories! Please see the solution branch if you have questions!

## The `rails routes` command

Here's a list of all of our application routes, indicating both the endpoint, the controller action, and a path prefix (i.e. the aliases we gave to our routes).

``` bash
rails routes
#
#    Prefix Verb   URI Pattern                Controller#Action
#    albums GET    /albums(.:format)          albums#index
#           POST   /albums(.:format)          albums#create
# new_album GET    /albums/new(.:format)      albums#new
#     album GET    /albums/:id(.:format)      albums#show
#
```

## Using View Helpers to Clean Up
If you're not using rails' built-in view helpers in your code, _you're working too hard_. Let's translate some of the "bad" code we wrote above into a cleaner, rails style:

#### `image_tag` view helper
Let's replace our `<img>` tag code above with the [`image_tag` view helper](http://apidock.com/rails/ActionView/Helpers/AssetTagHelper/image_tag):

``` html
<img src="<%= album.cover_art %>"> <!-- bad -->
<%= image_tag album.cover_art %>   <!-- good -->
```

Refresh, and view the source code for your page in your browser. What does the rails helper above look like once it's converted into normal HTML?

#### `link_to` view helper
Let's replace the `<a>` tag code above with the [rails-style `link_to` view helper](http://apidock.com/rails/ActionView/Helpers/AssetTagHelper/link_to):

``` html
<a href="/albums/new">Make a New Album</a>  <!-- bad -->

<%= link_to "/albums/#{album.id}" %>
<!-- or -->
<%= link_to album_path(album.id) %>
<!-- or  -->
<%= link_to album.id %>
<!-- or -->
<%= link_to album %>                        <!-- good -->
```

Refresh, and view the source code for your page in your browser. What does the rails helper above look like once it's converted into normal HTML?

## Stretch: Add Artists
Can you follow the steps outlined above to create `show` and `index` pages for musicians?
