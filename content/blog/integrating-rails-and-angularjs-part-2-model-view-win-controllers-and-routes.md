---
title: "Integrating Rails and AngularJS, Part 2: Model-View-Win, Controllers and Routes"
authors: ["gio"]
date: 2013-04-09T12:41:00+07:00
tags:
- rails
- angular
---

> MVC? MVP? MVVM? MV\*?  
> Screw all that.. Google wants you to call it: MVW!

<small>*Source code for application that we will create, can be accessed [here](https://github.com/giosakti/rails_angular_integration_example) (including part one)*</small>

Here's an interesting fact: most web framework nowadays declare themselves an MVC framework. However based on this [first note regarding MVC](http://heim.ifi.uio.no/~trygver/2007/MVC_Originals.pdf) by its conceptor, it's revealed that MVC aren't suited (or designed) for web application at all.

So how about Rails? isn't it also one of those MVC framework? Well the truth is Rails [closer](http://andrzejonsoftware.blogspot.com/2011/09/rails-is-not-mvc.html) to [Model2](http://en.wikipedia.org/wiki/Model2) than MVC itself. Although one can't deny that Model2 takes it root from MVC. This confusion has sometimes creates quite a stir at discussion boards or blog comment boards that is crowded by developers, which leads to [this](https://plus.google.com/+AngularJS/posts/aZNVhj355G2) narrative by Google's own Igor Minar whom develop AngularJS.

So **Model-View-Whatever** eh? Google? but why'd you still call controller in AngularJS as 'controller' ? not 'whatever' ? :D

IMO though 'whatever' is a bit long, so I want to call it **Model-View-Win** from now on...

## Does it matter? This MV thingy?

The word play isn't but the underlying concept is fundamental in shaping modern application development. The concept I'm talking about is **'separation of concerns'** where application is separated into several distinct sections, each manage a unique concern.

To make yourself even more grateful for the creation of this concept, now imagine a situation where a person whom not familiar with basic web application architecture have to create a complex PHP web application from scratch without using any framework. If they never heard of MVC, most probably the codes that they created are lots of PHP pages with **SQL query scattered all over** (I've seen this hell). After creating many functionalities for the web application, they resigned, you came in and have to create additional functionalities for the same application. Before accepting harsh reality that you had just resigned from Fortune 500 company just to enter company whom discourage the use of framework, you will realize that application without separation of concerns is very hard to understand and develop.

So, yeah, separation of concern is very important, but just using MV* framework is not enough to implement this philosophy. You also have to fully understand the framework intention to avoid putting code in the wrong section. Like putting too many boilerplates in controllers or putting too much logic in views makes Rails separation of concern useless.

## 'Controller' in Rails and AngularJS

This controller in Rails and AngularJS, what it'd do? Let's take a look at classical MVC controller definition:

> A controller is the link between a user and the system. It provides the user with input by arranging for relevant views to present themselves in appropriate places on the screen. It provides means for user output by presenting the user with menus or other means of giving commands and data. The controller receives such user output, translates it into the appropriate messages and pass these messages on to one or more of the views.

It really sounds so much different from what we use controller for nowadays! in Rails for example, controllers was never anything about 'link between user and the system', if anything, it was more about link between model and view (or templates). But that's not where the excitement ends, now we shall take a look at classical MVC view definition:

> A view is a (visual) representation of its model. It would ordinarily highlight certain attributes of the model and suppress others. It is thus acting as a presentation filter. A view is attached to its model (or model part) and gets the data necessary for the presentation from the model by asking questions. It may also update the model by sending appropriate messages. All these questions and messages have to be in the terminology of the model, the view will therefore have to know the semantics of the attributes of the model it represents. (It may, for example, ask for the model's identifier and expect an instance of Text, it may not assume that the model is of class Text.)

Sounds familiar? this is what we use controllers for in Rails! So this is where one of the difference lies then, controllers in rails IS views in classical MVC definition. Well obviously much has changed since the day of early GUI application that makes such paradigm-shift happen, the important thing that we can learn from this is that 'MVC framework' can be interpreted differently based on each framework design decision. However what matters most for us now is that by adhering to this principle, Rails has already separates the concern for you. link to the database? custom query? put it in model, custom data filtering? response manipulation? put it in controller. Let's take a look at typical Rails controller below:

```ruby
class UsersController -> ApplicationController
  def index
    @users = User.order('name ASC').limit(100)
  end
end
```

Above is a typical controller in Rails, where it manipulates model and make some variables readily available for view. Rails controller communicate with its view using instance variables `@users` and the model `User` is being manipulated by `order` and `limit` methods. Now how about controller in Angular? it's a bit different, let's take a look:

```
# app/assets/javascripts/app/controllers/session-controller.js.coffee

app.controller "SessionsController", ($scope, Session) ->
  $scope.session = Session.userSession

  $scope.create = ->
    if Session.signedOut
      $scope.session.$save().then ((response) ->
        $scope.$emit('event:loginConfirmed', response.data)
      ), (error) ->

  $scope.destroy = ->
    $scope.session.$destroy()
    $scope.$emit('event:logoutConfirmed')
```

That was my angularJS session controller. We would discuss about session in the next part, but for now pay attention at where the similarities and differences lies:

- Angular can't create 'instance variables' like rails, but they use `$scope` variable as a bridge between 'model' and 'templates'
- `$scope` is powerful, in examples above you can use `$scope` to communicate to parent controlller (using `$emit`) and children controller (using `$broadcast`)
- Angular controller can be injected with various factory-made or user-made services that makes it even more powerful, in this case `Session` is injected
- Both rails and angular controllers prepare variables and provide actions to be used by their respective views

<pre style="white-space: pre-wrap;">
<strong>Injected? Injection?</strong>

<small><em>Taken from <a href=http://en.wikipedia.org/wiki/Dependency_injection>wiki</a></em>

Dependency injection is a software design pattern that allows removing hard-coded dependencies and making it possible to change them, whether at run-time or compile-time.

in angular case, it means that you don't have to explicitly write your dependency but you just need to specify which service that you want to use in the parameters. Of course the service must be registered with the angular application before you can use it. For example in my `SessionsController` above, `Session` is a service that is injected by writing `Session` as one of `SessionsController` parameters.</small>
</pre>

Now, that was fun, we can continue building our rails-angular application then :)

## Continuing Where We Left Off

After all the preparation that we have done at part one, we can continue creates something that is more practical, a CRUD application. This CRUD application is actually an (ultimately) simple task management where you can create, read, update and destroy tasks. Let's begin from Rails by creating a **Task** model and migration.

```shell
rails g model Task title:string description:text
rake db:migrate
```

Don't forget to modify `database.yml` file accordingly as necessary. 

After that we can build API for the task by creating controller in Rails. For best practice purpose, I usually create an `api` namespace and create a base controller inside it so that any other controllers that is within the `api` namespace can inherit from base controller.

```shell
rails g controller api/base --skip-stylesheets --skip-javascripts
rails g controller api/tasks --skip-stylesheets --skip-javascripts
```

Below, you can see the example of a base controller, notice that it's actually a child of `ApplicationController`. Also notices that I set the `layout` to false and default `respond_to` to `:json`, both configuration will be inherited by base controller children.

```ruby
# app/controllers/api/base_controller.rb
class Api::BaseController < ApplicationController
  layout false
  respond_to :json
end
```

Replace the content of `TasksController` with this code below, observe that it's inherited from base controller that we created earlier and each actions returns `respond_with`. Don't forget to put `:api` namespace after `respond_with` to avoid rails `RoutingError` quirk.

```ruby
# app/controllers/api/tasks_controller.rb
class Api::TasksController < Api::BaseController
  def index
    respond_with :api, Task.all
  end

  def show
    respond_with :api, Task.find(params[:id])
  end

  def create
    respond_with :api, Task.create(params[:task])
  end

  def update
    respond_with :api, Task.update(params[:id], params[:task])
  end

  def destroy
    respond_with :api, Task.destroy(params[:id])
  end
end
```

Put routes information for `TasksController` in `config/routes.rb`

```ruby
# config/routes.rb
...
namespace :api do
  resources :tasks
end
...
```

You can test the API that you just created by accessing "api/tasks" from web browser, if its return something like `[]` then it's working. 

## Angular Configuration Files

With the `Task` API done, we can now move onto the angular side. Before continuing, you must create some angular configuration files first. Create `app/assets/javascripts/config` folder and create these following files (if you haven't done so already from part one):

```coffee
# app/assets/javascripts/config/csrf.js.coffee
# Useful for handling rails authenticity token.

app.config ($httpProvider) ->
  authToken = $("meta[name=\"csrf-token\"]").attr("content")
  $httpProvider.defaults.headers.common["X-CSRF-TOKEN"] = authToken

# app/assets/javascripts/config/push-state.js.coffee
# For activating HTML5 push state, 
# so that you don't have to use '/#/' in the url.

app.config ($locationProvider) ->
  $locationProvider.html5Mode true

# app/assets/javascripts/config/utf8.js.coffee
# For putting utf-8 in the request header

app.config ($httpProvider) ->
  $httpProvider.defaults.transformRequest.push (data, headersGetter) ->
    utf8_data = data
    unless angular.isUndefined(data)
      d = angular.fromJson(data)
      d["_utf8"] = "&#9731;"
      utf8_data = angular.toJson(d)
    utf8_data
```

Make sure that `config` folder is included by the `application.js` file. You must also include jquery now because `csrf.js.coffee` needs jquery to perform.

```javascript
// app/assets/javascripts/application.js

//= require jquery
//= require angular
...
//= require app/main
//= require_tree ./config
//= require_self
...
```

## ApplicationController and TasksController in Angular

Before creating `TasksController` in angular, I usually create `ApplicationController` first. Similar with its Rails counterpart, this controller is handy if you want all the other controllers to inherit from it. Although it's empty at the moment (I create it now solely for best practice purpose).

```
# app/assets/javascripts/app/controllers/application-controller.js.coffee
app.controller "ApplicationController", ($scope) ->
```

Again, don't forget to include the controller directory in `application.js`

```javascript
// app/assets/javascripts/application.js

...
//= require_tree ./config
//= require_tree ./app/controllers
//= require_self
...
```

Here comes the interesting part, the `TasksController` itself. Remember that from the beginning I try to model our angular application structure as close as possible with Rails, thus for this controller I try to make it RESTful.

```coffee
# app/assets/javascripts/app/controllers/tasks-controller.js.coffee

app.controller "TasksController", ($scope, $http, $location, $state, $stateParams) ->

  # =========================================================================
  # Initialize
  # =========================================================================

  $scope.tasks = {}
  if $state.current.name == 'tasks'
    $http.get("/api/tasks"

    # success
    ).then ((response) ->
      $scope.tasks = response.data
    
    # failure
    ), (error) ->

  $scope.task = {}
  if $state.current.name == 'edit'
    $http.get("/api/tasks/#{$stateParams['id']}"

    # success
    ).then ((response) ->
      $scope.task = response.data

    # failure
    ), (error) ->
...
```

The `$state` variable above is related to the angular routing system that we will discuss in the next section. The things that you must know from this code now, though, is that the application will load all tasks when the route points to `tasks`, which similar to `index` action in Rails and it will load a task for editing if an id is specified and the route points to `edit` (see `if $state.current.name...` above). Other than that, the application will define blank `$scope.tasks` and `$scope.task`.

For next portion of the code, we also must handle create, update and destroy action (new isn't necessary because it's only initialize the `$scope.task` to a blank hash, which has been done in the initialize code above).

```coffee
# app/assets/javascripts/app/controllers/tasks-controller.js.coffee
...
  # =========================================================================
  # Create
  # =========================================================================

  $scope.create = ->
    $http.post("/api/tasks",
      task:
        title: $scope.task.title
        description: $scope.task.description

    # success
    ).then ((response) ->
      $location.path "/tasks"

    # failure
    ), (error) ->

  # =========================================================================
  # Update
  # =========================================================================

  $scope.update = ->
    $http.put("/api/tasks/#{$scope.task.id}",
      task:
        title: $scope.task.title
        description: $scope.task.description

    # success
    ).then ((response) ->
      $location.path "/tasks"

    # failure
    ), (error) ->

  # =========================================================================
  # Destroy
  # =========================================================================

  $scope.destroy = (id) ->
    $http.delete("/api/tasks/#{id}"

    # success
    ).then ((response) ->
      $http.get("/api/tasks").then ((response) ->
        $scope.tasks = response.data
      ), (error) ->

    # failure
    ), (error) ->

  return false
```

The above code is interesting right? it's very similar with how rails handle things. But instead of manipulating model, the code above calls our `Task` API that was created earlier. It also handles redirection with `$location.path` following successful API calls.

Now after we create our controllers we can move on to the next step, routing and templates.

## Routes, Nested Routes, Templates

Routing is funny business in Angular, it's not weird or anything per se, but it's quite different especially if you have became very accustomed to rails routes. Angular ship with default routing functionality called the `$routeProvider` but I really dig the [ui-router](https://github.com/angular-ui/ui-router), which is also developed by angular team, so much that I never use `$routeProvider` from the beginning. The thing is, `ui-router` provide nested routing functionality that enables you to heavily customized your application layout and routing. For example, the `ui-router` enables you to change one or more portions of your page according to the current route, be it either footer, header, sidebar etc.

To utilize the `ui-router`, you have to first download it from [ui-router](https://github.com/angular-ui/ui-router) and after that putting it into the `vendor/assets/javascripts` folder. You must also include it in `application.js` and your angular initializer.

```javascript
// app/assets/javascripts/application.js
...
//= require angular
//= require angular-ui-states
//= require app/main
...
```

```coffee
# app/assets/javascripts/app/main.js.coffee

# Create 'app' angular application (module)
@app = angular.module("app", ["ui.compat"])
```

After you finished with `ui-router` configurations, you can create the routes file. As for me, I created it inside the `config` directory. Below is the first part of our routes configuration file.

```coffee
# app/assets/javascript/config/routes.js.coffee

# Configure 'app' routing. The $stateProvider and $urlRouterProvider
# will be automatically injected into the configurator.
app.config ($stateProvider, $urlRouterProvider) ->

  # Make sure that any other request beside one that is already defined
  # in stateProvider will be redirected to root.
  $urlRouterProvider
    .otherwise("/")

  # Define 'app' states
  $stateProvider
    .state "default",
      abstract: true
      views:
        "":
          controller: "ApplicationController"
          templateUrl: "/assets/layouts/default.html.erb"
```

The `ui-router` gives `$stateProvider` service that can be injected, as you may see above. But, I also utilize `$urlRouterProvider` to handle all 'invalid' route so that it will be redirected to "/". As for the `$stateProvider` itself, I started by creating an abstract route that gives default template, similar with `layouts/application.html.erb` in rails. You may notice that `ui-router` enables you to specify which controller and templateUrl to use in any particular route (or state in this case). This is hugely different from Rails where we only have capability to specify which controller to use for each route.

Oh, and abstract route means route that can't be accessed, but can be parent for other routes.

## templateUrl and ui-view

If you notice the previous `routes.js.coffee` code, you see that there is an attribute called `views`. This attribute is used for controlling specific part of html templates that have `ui-view` attribute in any of its tags. If the above code is using `""` after the `views`, it means that `ui-view` without any argument will be targeted. While on the other hand if the `views`, for example, is specified to `"test"` then it will target `< ANY ui-view="test" >` within the template.

Therefore, as for our application, we must specify tags with `ui-view` so that angular knows where to put our templates. Remember the `HomeController` and its `index` action in Rails that we created in part one? To continue with our application, we have to specify tag that contain `ui-view` so that default layout can be inserted in those tag.

```html
<!-- app/views/home/index.html.erb -->
<div id="wrapper" ui-view="">
</div>
```

We also have to create the default application layout.

```html
<!-- app/assets/templates/layouts/default.html.erb -->
<div ui-view=""></div>
```

## Task route and templates

We now focus on handling our task application routing and its respective templates. We can start by defining more routes in `routes.js.coffee` file for tasks.

```coffee
# app/assets/javascript/config/routes.js.coffee
...
    # Tasks
    .state "tasks",
      parent: "default"
      url: "/tasks"
      views:
        "":
          controller: "TasksController"
          templateUrl: "/assets/tasks/index.html.erb"

    .state "new",
      parent: "tasks"
      url: "/new"
      views:
        "@default":
          controller: "TasksController"
          templateUrl: "/assets/tasks/new.html.erb"

    .state "edit",
      parent: "tasks"
      url: "/:id/edit"
      views:
        "@default":
          controller: "TasksController"
          templateUrl: "/assets/tasks/edit.html.erb"
```

Why the `@default` you might ask? it's because the location of `ui-view` that you want to replace is located in the 'default' state, while 'new' and 'edit' state is grandchild of 'default'. This is the capability of `ui-router` that I'm talking about, that is to replace template in any location as you wish. By doing this, user will sense page transition to 'new' or 'edit' page, while actually it's just angular working behind the scenes, replacing the template as you specify. If you want to understand more regarding `ui-router` especially its nested routing & templating system, you may want to check its wiki [here](https://github.com/angular-ui/ui-router/wiki/Nested-States).

Next I will show you the templates for index, new and edit action. The \_form template is generic, so that it can be used by both new and edit action, just like Rails. We can include template within template by using the `ng-include` directive, see the new and edit templates below.

```html
<!-- app/assets/templates/tasks/index.html.erb -->
<ul>
  <li ng-repeat="task in tasks">
    Title: {{task.title}} <br/>
    Description: {{task.description}}
    <a href="/tasks/{{task.id}}/edit">Ubah</a>
    <a href="" ng-click="destroy(task.id)">Hapus</a>
  </li>
</ul>

<a href="/tasks/new">Add new task</a>


<!-- app/assets/templates/tasks/new.html.erb -->
<form ng-submit="create()">
  <ng-include src="'/assets/tasks/_form.html.erb'" />
</form>


<!-- app/assets/templates/tasks/edit.html.erb -->
<form ng-submit="update()">
  <ng-include src="'/assets/tasks/_form.html.erb'" />
</form>


<!-- app/assets/templates/tasks/_form.html.erb -->
<input type="hidden" ng-model="task.id" />

<label for="title">Title</label>
<input type="text" ng-model="task.title" />

<label for="description">Description</label>
<input type="text" ng-model="task.description" />

<button type="submit">Save</button> 
```

Lastly, to make our application works, we must specify **passthroughs** in rails so that every url request that was prefixed by `/tasks` are redirected to our angular application.

```ruby
# config/routes.rb
...
  # Passthrough to frontend
  match '/' => 'home#index'
  match '/tasks' => 'home#index'
  match '/tasks/*page' => 'home#index'
...
```

Now fire up your browser and ... ! (fill in the blanks).

## Conclusion

If you have some troubles or perhaps wants to take a look at my code, you can go to the address below:

**[Checkout the source here](https://github.com/giosakti/rails_angular_integration_example)**

We have discussed a lot of topics today, but our codes are still far from perfect. For example our controller can be simplified further if we take advantage of angular-resource. For that purpose, the next part will focus on angular service and angular resource. So, see you at the next part.

PS: next part will probably be delayed a week as my schedule is quite full next weekend.
