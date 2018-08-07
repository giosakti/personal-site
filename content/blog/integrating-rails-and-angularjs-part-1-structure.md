---
title: "Integrating Rails and Angularjs Part 1 Structure"
date: 2013-03-29T04:00:00+07:00
tags:
- rails
- angular
draft: true
---

<em>This is part one of several more parts regarding this topic. I will post at one-week interval for this series</em>

[lead_p]I'm a top-down person. So, whenever I try to understand or create a new concept, I like to climb my way up and see the views from above first before making any decisions. Therefore, when I first intended to create a single page application (SPA) with Angular as its front-end and Rails as its back-end, I'd love to talk about the <strong>structure</strong> first.[/lead_p]

[alert type="info"]
Source code for application that we will create, can be accessed <a href="https://github.com/giosakti/rails_angular_integration_example">here</a>
[/alert]

Before, I go any further, I want to explain on why I use Rails as an API back-end. You may want to argue that Rails isn't lightweight enough for that purpose and I agree, performance isn't exactly one of Rails strong suit. However, other than the obvious reason that my team is already familiar with it, Rails have some advantages that I find it hard to substitute.

First is the thousands (or tenth of thousands) gems, which is readily available in <a title="Rubygems" href="http://rubygems.org/" target="_blank">Rubygems</a>. I believe Rubygems is the perfect example of an excellent code-reuse platform. I love how easy it is to integrate my Rails application with those gems, which have significant effect in reducing boilerplate from my application. Oh, and have I mention how vibrant the community that supported those gems?

Second, I like the "Rails-way". I know how opinionated it is, but that's OK with me because I tend to agree with most of its opinion. Although, I can relate to people whom disagree, because they will find it very hard to use Rails and then they have to customize the hell out of it. Well as for me, it enables me to "write less for more" and due to my <a title="Repetitive Strain Injury" href="http://en.wikipedia.org/wiki/Repetitive_strain_injury" target="_blank">RSI</a>, less for more is very important.

<!--more-->

<h2>Starting new Rails application</h2>

Ok, enough with the chit chat, now we can continue. First, let's observe initial Rails application structure created by the <code>rails new</code> command.

<pre class="theme:github toolbar:2 tab-size:2 decode:true">rails new project_one</pre>
<pre class="nums:false lang:vim mark:8 decode:true">
project_one
-- app
---- assets
---- controllers
---- helpers
---- mailers
---- models
---- views
-- config
-- db
-- doc
-- lib
-- log
-- public
-- script
-- test
-- tmp
-- vendor
</pre>

Rails is a complete web application framework with MVC pattern, thus naturally it has their own templating system. Rails templating system uses .erb format by default, but you can customize it easily with other "aftermarket" solutions, such as the 'cute' <a href="http://haml.info">.haml</a>. Templates go into 'views' folder that you can see above (I believe view from V in MVC for Rails is not the same with this views folder, but let's get to it sometimes later). 

If we want to use Angular and create SPA, Rails templating system won't be used much because Rails as an API-backend will (mostly) only spout JSON. Therefore I usually only fill this 'views' folder with default application layout and one 'blank' page. With the exception of situations where I have to provide some complex JSON formats and have to use <a href="https://github.com/nesquena/rabl">rabl</a> or <a href="https://github.com/rails/jbuilder">jbuilder</a> gems.

This 'blank' page mentioned above corresponds with one controller that will serve the Angular application landing page. Let's create it inside the project that we just made.

<pre class="theme:github toolbar:2 tab-size:2 decode:true">
rails g controller home index --skip-stylesheets --skip-javascripts
</pre>

You may notice that I use the options <code>--skip-stylesheets --skip-javascripts</code>, I'm doing this because I'm not keen on how Rails structure my stylesheets and I also skip Rails generated javascripts because I want to avoid conflict with Angular js files. Having said that, now we can do some initial configuration:

<ul><li><strong>Modify root route to the index action within home controller</strong>, which we have just created.</li></ul>

<pre class="nums:false lang:ruby decode:true" title="config/routes.rb">
ProjectOne::Application.routes.draw do
  root to: 'home#index'
end
</pre>

<ul><li>Don't forget to <strong>remove the default index.html in public.</strong></li></ul>

<pre class="theme:github toolbar:2 tab-size:2 decode:true">
rm public/index.html
</pre>

<ul><li><strong>Modify the default application.html.erb file that will house our Angular application.</strong> You may want to take particular attention at the opening html tag, because now it contains <code>ng-app="app"</code>, which will tell Angular to serve an "app" application here.</li></ul>

[alert type="info"]
Angular application can be placed almost anywhere and this is the beauty of it. You can "plug-and-play" an angular application to your existing web application using this directive inside any html tag <code>ng-app="nameOfYourApplication"</code>
[/alert]

<pre class="nums:false lang:html decode:true" title="app/views/layouts/application.html.erb">
<!DOCTYPE html>
<html ng-app="app">
  <head>
    <title>ProjectOne</title>
    <%= stylesheet_link_tag    "application", :media => "all" %>
    <%= javascript_include_tag "application" %>
    <%= csrf_meta_tags %>
  </head>
  <body>
    <%= yield %>
  </body>
</html>
</pre>

<ul><li>Now, <strong>make sure that the controller index template</strong>, which will be yielded in <code><%= yield %></code> above is <strong>available and empty.</strong></li></ul>

<pre class="theme:github toolbar:2 tab-size:2 decode:true">
cat > app/views/home/index.html.erb
</pre>

don't forget to press ctrl+c immediately after typing the above command.

<h2>Including Angular Libraries</h2>

Next, we have to include Angular libraries into our Rails application. We can use gems for this purpose, but I prefer to include the corresponding js files manually and taking advantage of Rails asset pipeline. Asset pipeline is a system within Rails that 'automagically' combines and compiles all assets into browser-readable formats (such as .erb or .haml to html) during deployment process so that it can be opened in browser effortlessly. You can put your assets into any of three locations that was created by default. This is taken straight from their documentation:

[alert type="info"]
<code>app/assets</code> is for assets that are owned by the application, such as custom images, JavaScript files or stylesheets.<br/>
<code>lib/assets</code> is for your own libraries’ code that doesn’t really fit into the scope of the application or those libraries which are shared across applications.<br/>
<code>vendor/assets</code> is for assets that are owned by outside entities, such as code for JavaScript plugins and CSS frameworks.
[/alert]

Rails asset pipeline is activated by default, but to make sure you can check it out at the <code>config/application.rb</code> file.

<pre class="nums:false lang:ruby decode:true" title="config/application.rb">
module ProjectOne
  class Application < Rails::Application
    ...
    # Enable the asset pipeline
    config.assets.enabled = true
    ...
  end
end
</pre>

So let's download latest angular library and put it inside the <code>vendor/assets/javascripts</code> folder:

<pre class="theme:github toolbar:2 tab-size:2 lang:sh decode:true">
curl -o vendor/assets/javascripts/angular.js http://code.angularjs.org/1.0.5/angular.js
</pre>

As of today, Angular has stable version of 1.0.5. You can check the latest version <a href="http://angularjs.org">here</a> and update the above command as necessary. I also prefer to download the uncompressed files for easier debugging whilst Rails can helps in minifying the files later during deployment.

However, only putting the latest angular js files won't make it work, so let's modify the manifest file that list all js files that will be included in the Rails asset pipeline (if you don't specify it here, it won't be included by Rails). Rails already created this file and fill it with some initial configuration, but I prefer to replace it with my own configuration. 

<pre class="lang:js decode:true " title="app/assets/javascripts/application.js" >
//= require angular
//= require_self
</pre>

As you can see above, I don't think that I need jquery for now, so I remove it. <code>require_tree .</code> is also a bit tricky so I left it out and replace it with require_self.

<h2>Initializing Angular Application</h2>

To initialize <code>ng-app="app"</code> that written on our html block, we have to activate it using <code>angular.module</code>. Before doing this I created a special folder within the <code>assets/javascripts</code> folder that will contains all of my angular-related codes:

<pre class="nums:false lang:vim mark:9 decode:true">
assets
-- javascripts
---- app
------ controllers
------ directives
------ filters
------ resources
------ services
------ main.js.coffee
---- config
---- application.js
</pre>

I will give detailed explanation regarding the structure later, but for now we will put <code>angular.module</code> within <code>main.js.coffee</code> file. This file will be the starting point of our angular application.

<pre class="lang:coffee decode:true " title="app/assets/javascripts/app/main.js.coffee" >
# Create 'app' angular application (module)
@app = angular.module("app", [])
</pre> 

<br/>
<h3>Additional Steps (Optional)</h3>

You may want to include some additional gems that relate to assets, such as support for .haml, .sass and coffeescripts format. I provide you with an example of how my assets group looked like within the <code>Gemfile</code> below:

<pre class="nums:false lang:ruby decode:true" title="Gemfile">
group :assets do
  # HTML & CSS replacement
  gem 'haml-rails', '~> 0.4'
  gem 'sass-rails', '~> 3.2.6'

  # JS replacement
  gem 'coffee-rails', '~> 3.2.2'

  # JS interpreter
  gem 'therubyracer', '~> 0.11.3', platform: :ruby
  gem 'therubyrhino', '~> 2.0.2', platform: :jruby

  # Misc
  gem 'uglifier', '~> 1.3.0'    # wrapper for UglifyJS JavaScript compressor
end
</pre>

Don't forget to <code>bundle</code> after every <code>Gemfile</code> modification.

<pre class="theme:github toolbar:2 tab-size:2 lang:sh decode:true">
bundle
</pre>

<h2>Structure</h2>

Looking back at the structure, I tried to model it as close as possible to Rails structure. 

<pre class="nums:false lang:vim mark:9 decode:true">
assets
-- javascripts
---- app
------ controllers
------ directives
------ filters
------ resources
------ services
------ main.js.coffee
---- config
---- application.js
</pre>

First I want to explain the 'config' folder, this folder contains all configurations and initializers. Some of examples on what I put in this folder is the <code>constants.js.coffee</code> file. This file contains application-wide constants to be used by the angular application. I also have <code>debug.js.coffee</code> file which stores some variables for easier debugging. Also, don't forget the all-important <code>routes.js.coffee</code> that stores the routing of our client-side application.

<pre class="lang:coffee decode:true " title="app/assets/javascripts/config/constants.js.coffee" >
# Prefix url string for api calls
app.constant "apiPrefix", "/api"

# Location for session information within sessionStorage
# Related to authentication & session, will get back to this later
app.constant "sessionStore", "_starqle_session"
</pre>

<pre class="lang:coffee decode:true " title="app/assets/javascripts/config/debug.js.coffee" >
app.run ($rootScope, $state, $stateParams) ->
  # You can turn this off on production.
  $rootScope.$debugMode = "on" # "off"

  # Capture current state and stateParams, this variable can be showed
  # in browser for debug purpose.
  $rootScope.$state = $state
  $rootScope.$stateParams = $stateParams
</pre>

You may notice the $state variable in <code>debug.js.coffee</code> above, this code is related to routing and I will get back to it later on part 2. I will also gives example on my implementation of <code>route.js.coffee</code> there.

And if you use Rails, you must not forget to automatically set csrf authenticity token in all requests so that the request won't be rejected by Rails. I also put this <code>csrf.js.coffee</code> inside <code>config</code> folder.

<pre class="lang:coffee decode:true " title="app/assets/javascripts/config/csrf.js.coffee" >
app.config ($httpProvider) ->
  authToken = $("meta[name=\"csrf-token\"]").attr("content")
  $httpProvider.defaults.headers.common["X-CSRF-TOKEN"] = authToken
</pre>

Other than <code>config</code> folder, there is <code>controllers</code> folder. This folder contains, well, controllers for our angular application. There is also <code>directives</code> folder, directive is an important concept in angular that makes it possible to implement a real "unobtrusive javascript". Directive are custom attributes that can be placed within any html tags and its usage is controlled by separated file located in this <code>directives</code> folder. The ability to place custom attributes that can be controlled in separated files gives our html template 'steroid' that makes them more powerful in web application context, while keeping it uncluttered. Angular comes with many home-ground directives that you may find interesting, you can check it out <a href="http://docs.angularjs.org/api/">here</a>.

Next is <code>filters</code> directory in which you can put your custom filters here and then there are also <code>resources</code> and <code>services</code> folders. You can see resource on angular like model in Rails, the difference lies on its datasource. If Rails application model glue the application with database, Angular resource is the interface between the application itself and external API that will be used as the datasource (which, in this case, Rails that spout JSON). Lastly, I see service as 'global' function that can be injected and used anywhere within the angular application.

<h2>Conclusion</h2>

To make sure that everything is working, you can just start your rails server and point your browser to the application url. If you see something like this when you see your page source using firebug...

<pre class="lang:html decode:true">
...
<html class="ng-scope" ng-app="app">
...
</pre>

... the <code>class="ng-scope"</code> means that your angular application has been initialized.

Ok, that concludes the first part, as you can see up to this point this writeup is (perhaps) not very practical. But practical is not my intention from the beginning, because I want to take different angle on explaining things, such as talking a bit more about concept and architecture. So let's continue with part 2 next week :)

[alert type="info"]
Source code for application that we have created, can be accessed <a href="https://github.com/giosakti/rails_angular_integration_example">here</a>
[/alert]
