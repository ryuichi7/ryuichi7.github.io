---
layout: page
#
# Content
#
subheadline: "Rails and Javascript"
header: false
title: "Adding Javascript to my first rails app"
teaser: "See some of the asynchronous functionality I added to my first rails app"
categories:
  - blog	
#tags:
#  - 
---

My first rails application, humorously titled Tummy Topics, features a way for users to list their favorite recipes as well as rate and comment on other user's recipes.

Through the use of the popular gems, Devise and CanCan, as well as setting up some complex model associations, I was able to create a fairly functional application that possessed the authentication, authorization and overall features I desired of this basic app.

There was still something missing though. Asynchronous functionality. Whenever a comment was written, a rating was given, or a recipe was edited, a new page would have to be loaded. I wanted a more seamless user experience and therefore would have to utilize javscript to manipulate the DOM (Document Object Model).

## Comments on the fly

The first Asynchronous functionality I wanted to introduce was the ability to create a comment and see it appended to the DOM immediately after submission. I conciously decided to use the rails pattern, 'data-remote=true', to create this funtionality. When you add the data-remote=true attribute to a rails form or link it submits an AJAX request via the jQuery-UJS (unobstrusive javascript) library and prevents the usual page reload of a request. You can view the source for jQuery-UJS [here](https://github.com/rails/jquery-ujs/blob/master/src/rails.js). I wanted to try to employ this convention as it's a much cleaner way of making an AJAX request. Rather than adding a script like this to your view:

{% highlight js%}
$('form').on('submit', function (e) {
	e.preventDefault();
	var values = $(this).serialize()
	$.post('/exampleUrl', values).done(function(data) {
	// some response code here
	});
})
{% endhighlight js%}
Note: the [jQuery.serialize()](https://api.jquery.com/serialize/) method changes a form element into a URL-encoded string.


You can simply add this to your html form element:

{% highlight html%}
<form data-remote='true' action='/exampleurl' > 
{% endhighlight%}

The implementation in my Tummy Topics app is as follows:

{% highlight erb %}
<div class="panel-body">
	<%= form_for @recipe.comments.build, remote: true, action: comments_path, method: 'post' do |f| %>
		<%= f.hidden_field :recipe_id, value: @recipe.id %>
		<div class="form-group">
			<%= f.label "Add a comment" %>
			<%= f.text_area :content, cols: 55, rows: 5, class: "form-control" %>
			<span class="help-block"></span>
		</div>
		<%= f.submit class: "btn btn-primary" %>
	<% end %>
</div>
{% endhighlight %}

Rails.js has a script that sets up a listener that responds to the data-remote=true tag and sends an AJAX request to the desired action, in this case `action: /comments` with a `method: POST`. The request is made via Javascript format. Rails leaves it up to you how you'd like to deal with the response, but since javaScript objects can't be rendered in html format, we need to create a format to handle the response.

{% highlight ruby %}
# controllers/comments_controller.rb

def create
	@comment = current_user.comments.build(comment_params)
	
	respond_to do |f|
		if @comment.save
			f.html { redirect_to recipe_path(@comment.recipe)}
			f.js {}
		else
			f.html { redirect_to :back, alert: "please properly fill in comment field" }
			f.js { render json: @comment.errors, status: :unprocessable_entity }
		end
	end
end

{% endhighlight %}

Here we use the Rails' `respond_to` block which allows responses to be made with different formats. Conventionally Rails responds with html unless otherwise instructed.

You'll notice the curly braces following the js format in the `@comment.save` section of the `respond_to` block. This makes use of another rails convention which looks for a controller action's corresponding view. Instead of explicity having to tell Rails what template to render like this:

{% highlight ruby %}
# controllers/comments_controller.rb
f.js { action: :create, status: :created, location: @comment }
{% endhighlight %}


We rely on the defaults and create a new corresponding view in the `views/comments` folder titled `create.js.erb`. In `create.js.erb` we can use javaScript to handle the server response:

{% highlight js %}
// views/comments/create.js.erb

html = '<%= j (render @comment) %>'

$('div#comments_display').append(html);
$('#new_comment')[0].reset();
$("#dialog").dialog('open');
{% endhighlight %}

If you're wondering what `j` is, it's an alias for the `escape_javascript` method, which escapes javascript characters so that the corresponding `_comments.html.erb` template being rendered can be properly processed by the html file it's being inserted into.

Note: The [$.dialog()](http://api.jqueryui.com/dialog/) function is a part of the jQuery-UI library and dialog widget. It renders an alert box that can be nicely formatted and comes with some handy built-in features.

Now that I had the ability to save comments to the database and append them to the DOM with ever having to reload the page, I knew there were other parts of my app where I wanted to add the same logic.

## Adding ingredients to a recipe

In my original version of the Tummy Topics app, a user was instructed to enter each ingredient into a text box that looked something like this:

![recipe ingredients image]({{ site.urlimg }}ingredient_shot.png)

Then the custom method `ingredients_attributes=()` I made would parse the input and insert a new row into the `recipe_ingredients` table while creating a new row in the `ingredients` table if it didn't already exist.

{% highlight ruby %}
def ingredient_attributes=(ingredient_attributes)
	ingredient_attributes[:ingredients].split(/\r\n/).each do |ingredient|
		
		x = ingredient.split("-")
		measurment = x.first.strip
		ingr_name = x.last.strip
		
		i = Ingredient.find_or_create_by(name: ingr_name.downcase.singularize)
		
		if !self.ingredients.include?(i)
			self.recipe_ingredients.build(ingredient: i, measurement: measurment, name: ingr_name)
		end
	end
end
{% endhighlight %}

This worked well enough, but left lots of room for error if the directions for adding a '-' and skipping a line weren't followed explicitly by the user. I therefore wanted a way to add ingredient fields as needed. As I've intended to use this app as a way for me to learn, I decided to use plain jQuery rather than Rails' data-remote='true' pattern. To achieve this, I needed to create a separate template and route to be able to pull from and append to my `recipes/new.html.erb` file. I created a button:

{% highlight html %}
<!-- recipes/_form.html.erb  #the partial referenced in my recipes/new.html.erb file -->
<button class='btn btn-primary' id='button' data-url="/recipes/ingredients/new">add another ingredient</button>
{% endhighlight %}

and a template `ingredients/new.html.erb` with the raw html I wanted to append to the `recipes/new` page:

{% highlight html %}
<!-- ingredients/new.html.erb -->
<input class="form-control comment col-md-6" type="text" name="recipe[ingredients_attributes][name]" id="recipe_ingredients_attributes_name" placeholder="Name">
<input class="form-control comment col-md-6 col-md-offset-1" type="text" name="recipe[ingredients_attributes][measurement]" id="recipe_ingredients_attributes_measurement" placeholder="Measurement">
{% endhighlight %}

Using the data attribute introduced in HTML5 is was able to store arbitrary data in the button to be referenced within the jQuery call I would be making to the server like so:

{% highlight js %}
$("button#button").click(function(e) {
	$.get($(this).data('url'), function(response) {
		$('#new-form').append(response);
	});
	e.preventDefault();
});
{% endhighlight %}

This AJAX get request to the route `/recipes/ingredients/new` grabs the raw html stored in the `ingredients/new/html.erb` template and appends it to the DOM, creating a new field to enter an ingredient and measurement into.

Note: In the `ingredients_controller` I made sure to add `layout: false` to the `render` method in the `new` route to make sure the only code I was grabbing was the raw html stored in the `ingredients/new.html.erb` template.

{% highlight ruby %}
# ingredients_controller.rb
def new
	render :new, layout: false
end
{% endhighlight %}

Lastly, I had to change some of my code to account for multiple fields. Using the Rails `fields_for` form helper and a link to the `ingredients/_fields.html.erb` partial, I was able to achieve this. Rails' `fields_for` usually handles the creation of nested forms for you, but in this case, because I was customly adding raw html to the DOM, I had to tell erb directly to store each set of inputs in an array within the params hash being passed to the associated controller upon form submission. So within the input fields I added a `[]` so that erb would recognize more than one set of inputs.

{% highlight html %}
<!-- ingredients/new.html.erb -->
<input class="form-control comment col-md-6" type="text" name="recipe[ingredients_attributes][name]" id="recipe_ingredients_attributes_name" placeholder="Name">
{% endhighlight %}

became:

{% highlight html %}
<!-- ingredients/new.html.erb -->
<input class="form-control comment col-md-6" type="text" name="recipe[ingredients_attributes][][name]" id="recipe_ingredients_attributes_name" placeholder="Name">
{% endhighlight %}

Now if I added multiple fields to the DOM using using javascript, the new fields would still be recognized by Rails and stored correctly in the params hash.

I still had to handle a few more things. Firstly, I needed to white list my `ingredients_attributes` attribute for mass assignment like so:

{% highlight ruby %}
# controllers/recipes_controller.rb
def recipe_params
	params.require(:recipe).permit(:name, :user_id, :description, :directions, :image, ingredients_attributes: [:name, :measurement])
end
{% endhighlight %}

and I needed to make a few changes to the `ingredients_attributes=()` method in my recipes model:

{% highlight ruby %}
# models/recipes.rb
def ingredients_attributes=(ingredients)
		recipe_ingredients.destroy_all if persisted?

		ingredients.each do |ingredient|
			unless ingredient[:name].blank?
				ingr_name = ingredient[:name]
				measurement = ingredient[:measurement]

				i = Ingredient.find_or_create_by(name: ingr_name.downcase.singularize)
					
				if !ingredients.include?(i)
					recipe_ingredients.build(ingredient: i, measurement: measurement, name: ingr_name)
				end
			end
		end
	end
{% endhighlight %}

After all this, I was able to add new ingredient fields with one click and leave less room for user error:

![ingredients-fields image]({{ site.urlimg }}ingredient-fields.png)

## Scroll and Load

To further add to user experience, I wanted to be able to load recipes on the main index page without having to paginate or click on a link to load more. If this site were ever to actually be populated with a large number of recipes, the index page would take far too long loading the whole list of recipes in one request. 

Once again, AJAX to the rescue. I wanted a user to see 9 recipes initially and then be able to scroll to the bottom of the page and have 9 more loaded and so on. To achieve this, I knew I'd have to send a data element to be processed by the controller at which point it could query the database and return the desired amount of recipes.

Before I could do anything though, I realized I'd have to make objects on the client side to be properly parsed and appended to the dom. First, I set up a recipe object that mirrored the recipe model I already had set up in Rails:

{% highlight js %}
// javascripts/recipes.js
function Recipe(id, name, directions, description, user, postDate, comments = null, ratings = null, image) {
	this.id = id
	this.name = name
	this.directions = directions
	this.description = description
	this.user = user
	this.postDate = postDate
	this.comments = comments
	this.ratings = ratings
	this.image = image
}
{% endhighlight %}
Note: notice `comments = null` and `ratings = null`. Settings null options to be passed to an object is a new feature of ECMAScript6.

Then I set up serializers using the `active_model_serializers` gem so that I could retrieve serialized json from the server.

{% highlight ruby %}
# serializers/recipe_serializer.rb
class RecipeSerializer < ActiveModel::Serializer
  attributes :id, :name, :description, :directions, :post_date, :image
  has_many :ratings
  has_many :comments
  has_one :user

  def image
  	object.image.url.tap do |image|
  		if image.match(/default_recipe_image|placeholder_image/)
  			return "/assets/#{image}"
  		end
  	end
  end
end
{% endhighlight %}
Note: I had to set up serializers for my other models to be able to associate them to the recipe serializer which I didn't explicitly show here. You also have the ability to set up custom methods that you can pass to the `attributes` method for serialization, which you can see in my `image` method.

Then I added a `respond_to` block to my `recipes_controller` to handle the json response:

{% highlight ruby %}
# controllers/recipes_controller.rb
def index
	
	...

	respond_to do |f|
		f.html { render :layout => 'recipe_index' }
	  f.json { render json: @recipes }
	end
end
{% endhighlight %}

which resulted in a json object that I'd be able to pull data from:

{% highlight js %}
// recipes.json
{
	id: 33,
	name: "Chicken stew",
	description: "This is my favorite chicken stew recipe. It's been in the family for years and I really love it!",
	directions: "roast the chicken cut the vegetables cook it up!",
	post_date: "05/06/2016",
	image: "/assets/original/default_recipe_image.jpg",
	ratings: [
		{
			id: 53,
			score: 3
		}
	],
	comments: [ ],
	user: {
		id: 11,
		email: "brett@mail.com",
		thumb: "/system/users/avatars/000/000/011/thumb/brett.jpeg?1463692192",
		email_name: "Brett"
	}
}
{% endhighlight %}

Now that I had a properly formatted json object to pull from, I needed to create some functions that would be able to grab the data from the json response, format it, and properly append it to the DOM. 

The most difficult thing to figure out was how to format the response and be able to make it look exactly like the recipe objects that already existed in the DOM. Since these were formatted using Rails methods and erb, I had to find a way to create a template that would mirror these objects. I decided to use `Handlebars.js` which is a great formatting tool. I won't go into it in too much detail here, but it lets you dynamically add variables to html in a similar way erb allows you to add Ruby variables to your views.

So I created a template to pull from in my `recipes/index.html.erb` file:



{% highlight html %}
{% raw %}
	<!-- handlebars template -->
	<!-- views/recipes/index.html.erb -->

	<div class="col-md-4">
	  <div class="col-md-12 recipe-index">
	  	<div class='image-hover'>	
				<a href="/recipes/{{recipe.id}}">
					<img class="index-image" title="" data-toggle="popover" data-trigger="hover" data-placement="auto left" data-content="{{recipe.description}}" data-original-title="{{name}}" src="{{recipe.image}}" alt="recipe image">
				</a>
			</div>
			<p></p>
			<strong><a href="/recipes/{{recipe.id}}">{{name}}</a></strong>
			<p></p>
	    <h6>Posted on: {{recipe.postDate}}</h6>
			{{#if rating}}
				<div class="rating-box"><div class="display-rating" data-rating={{rating}}></div> ({{ratingLength}})</div>
			{{/if}}
			{{#if recipe.comments}}
				<button name="button" type="submit" class="see-comments btn btn-default collapsed" data-id="{{recipe.id}}" data-toggle="collapse" data-target="#collapse{{recipe.id}}" aria-expanded="false">See Comments</button>
				<div class="collapse" id="collapse{{recipe.id}}">
					{{#each recipe.comments}}
						<div class='comment-well'>
							<p>{{this.content}}</p>
							<h6>Posted on: {{this.post_date}}</h6>
							<div class="recipe-profile-card">
								<a href="/users/{{this.commenter.id}}"><img class="img-circle" src="{{this.commenter.thumb}}" alt="User thumb"></a>
								By: <a href="/users/{{this.commenter.id}}">{{this.commenter.email_name}}</a>
							</div>
						</div>
					{{/each}}
				</div>
			{{/if}}
			<div class="recipe-profile-card">
				<a href="/users/{{user.id}}"><img class="img-circle" src="{{user.thumb}}" alt="User thumb"></a>
				Recipe by: <a href="/users/{{user.id}}">{{user.email_name}}</a>
			</div>
	  </div>
	</div>

{% endraw %}
{% endhighlight %}

Note: The double curly braces denote Handlebars syntax that act as variables or boolean/iteration blocks. You can read the handlebars docs [here](http://handlebarsjs.com/)

So in my `recipe.js` file I set up functions to format and append the new recipe object I created from each json response to the DOM:

{% highlight js %}
// javascripts/recipes.js

// create new recipe object from json
function createRecipe(recipe) {
	var newRecipe = new Recipe(
				recipe.id,
				recipe.name,
				recipe.directions,
				recipe.description,
				recipe.user,
				recipe.post_date,
				recipe.comments,
				recipe.ratings,
				recipe.image
	);
	return newRecipe;
}

// format the new recipe object to be inserted into template. This includes assigning key/value pairs to be used within the handlebars template
// I created the openRow and closeRow variables for bootstrap formatting. This allowed me to dynamically set rows of three for displaying the recipes.
function formatForTemplate(recipe, openRow = false, closeRow = false) {
	var values = {
		name: recipe.name.titleize(),
		recipe: recipe,
		user: recipe.user
	};

	if (recipe.ratings.length > 0) {
		values["rating"] = recipe.ratingAvg();
		values["ratingLength"] = recipe.ratings.length;
	};
	if (openRow === true) {
		values["open"] = true;
	};
	if (closeRow === true) {
		values["close"] = true;
	};
	return values;
};

// take all recipes from json response, parse them, and append them to the DOM via a handlebars template
function parseAndDisplay(recipes) {
	inGroupsOf(recipes, 3).forEach(function(group) {
		var source = $("#recipe-template").html();
		var template = Handlebars.compile(source);
		var recipeRow = '<div class="row">';
		group.forEach(function(recipe) {
			if (recipe !== null) {
				var rec = createRecipe(recipe);

				recipeRow += template(formatForTemplate(rec));
			}
		});
		recipeRow += '</div>';
		$('#recipes').append(recipeRow);
		displayRating();
	});
}
{% endhighlight %}

Then I created a script to request 9 recipes at a time when scrolling to the bottom of the page:

{% highlight js %}
// recipes.index.html.erb

// script in the recipes/index view that requests 9 recipe objects upon scrolling to the bottom of the page and then appends them to the DOM
var data = { limit: 0 };
$(window).scroll(function() {
	if ($('#recipes').data('is-searched') === false) {
    if($(window).scrollTop() + $(window).height() == $(document).height()) {
     	data["limit"] += 9;
      getMoreRecipes(data);
    }
  };
});


function getMoreRecipes(data) {	
	$.get('/recipes.json', data, function(response) {
		var recipes = response.recipes;
		parseAndDisplay(recipes);
	});	
}
{% endhighlight %}

and the corresponding call in the `recipes_controller`

{% highlight ruby %}
# controllers/recipes_controller.rb
def index
	if params[:user_id]
		@recipes = User.find(params[:user_id]).recipes.alphabetized
	else
		@recipes = Recipe.alphabetized.limit(9).offset(params[:limit])
	end
	respond_to do |f|
		f.html { render :layout => 'recipe_index' }
	  f.json { render json: @recipes }
	end
end
{% end highlight %}

This created a much smoother user experience. Instead of having to paginate, the user simply scrolls to the bottom and the next set of recipes appears on the page.

## Errors

I found the most difficult part of using AJAX calls to be the error handling. As Rails usually does this for you by adding a `.field_with_errors` class to the form and appending error messages for you, I had to figure out how to do this with plain javaScript calls. 

After reading a few tutorials, I found the best way to do this was to set up a universal event listener pertaining to the javaScript events fired during an AJAX call. In the event of an error, the `ajaxError` event would be triggered and I'd be able to parse the response and append it to the form. In the case of creating a comment I added this event listener:

{% highlight js %}
// javascripts/recipes.js

//look for ajax response. if success fire event clear error message if any, if error, fire event and append error message to form
$(document).bind('ajaxSuccess','form#new_comment', function(event, xhr, settings) {
	if (settings.url === '/comments') {
		$('.form-group.has-error').each(function(){
	    $('.help-block', $(this)).html('');
	    $(this).removeClass('has-error');
	  });
	}
})
.bind('ajaxError','form#new_comment', function(event, xhr, settings) {
	if (settings.url === '/comments') {
		$('.alert').remove();
		if (xhr.status === 500) {
			$("#comment_form").children('.panel-body').prepend('<div class="alert alert-danger alert-dismissible" role="alert"><button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>Please sign in to create a comment!</div>');
		} else {
			$('textarea#comment_content').closest(".form-group").addClass('has-error')
			.find('.help-block').html($.parseJSON(xhr.responseText).content);
		};
	}
})
{% endhighlight %}

I believe in rails you can bind directly to the events triggered via jQuery-UJS. Hence, 

{% highlight js %}
$(document).bind('ajaxError','form#new_comment', function(event, xhr, settings) {}); 
{% endhighlight %}

could become

{% highlight js %}
$('form#new_comment').bind('ajax:error', function(event, xhr, settings) {})`
{% endhighlight %} 

but in this case the first version worked well for me, so I stuck with it.

## Thoughts and TODOs

This was an immensly helpful project that really expanded my knowledge of javaScript and how to add asynchronous functionality to an application. I've gained a new respect and interest for javaScript and hope to learn more ways to incorporate it's functionality into my projects. 

I still feel that my app has a few things left to be improved. Firstly, I was able to create functionality for lazily loading recipes on the index page, but I ran into an issue when using the search function within the index page. I had to insert a data attribute that would keep arbitrary recipes from being loaded upon page scroll after a certain search query was made. For now this works because I have so few recipes to search through, but if there were a much larger dataset, I'd want to create functionality that allowed for lazily loading results of a query as well, rather than just appending all the results at once.

I also want to improve the search functionality I created. It's not very accurate and I'd love to incorporate auto-complete functionality for a better user experience. I tried to use the rails `auto-complete` gem but had difficulty getting the results to render correctly due to the way I have my associations set up.

Creating this app was a great experience and I can't wait to work more with javaScript and it's exciting libraries and frameworks!
















