---
title: "Integrating Rails and AngularJS, Part 3: REST and Angular Resource"
authors: ["gio"]
date: 2013-05-31T21:58:00+07:00
tags:
- rails
- angular
---

> `$resource`  
> The best way for our Angular app to communicate with RESTful APIs

<small>*Source code for application that we will create, can be accessed [here](https://github.com/giosakti/rails_angular_integration_example) (including every part before)*</small>

Initially, I intended to merge topic about Angular `$resource` and Service into one part. However I've come to realize that both topics are quite different and needed to be separated. Therefore in this part I will exclusively discuss about `$resource` first and how it can helps us in communicating with RESTful API.

## Let's take a REST

I realize that majority of (good) programmers are witty. I like on how they made pun on things like technological abbreviation and such, like REST, which we will discuss and those "stupid", recursive abbreviation like GNU. Perhaps because tinkering with code everyday, constructing application on your mind before typing (every good programmer does this) finally making an effect to your body and soul. 

Putting that aside, REST (REpresentational State Transfer‎) is an exceptionally important concept in web application development. It's one of dominant communication protocol for distributed system (the other one being [SOAP](https://en.wikipedia.org/wiki/SOAP)). I won't talk much about it here, because I learned it several years ago from an excellent and comprehensive article (with yet another witty title :D) that you can also read to understand it clearly - [How I Explained REST to My Wife](https://web.archive.org/web/20130116005443/http://tomayko.com/writings/rest-to-my-wife).

<pre style="white-space: pre-wrap;">
<small>During my writing of this article, I just realized that the aforementioned article has been put down by it's original writer, Ryan Tomayko, due to request from people that find the title offensive. However, after thinking about it carefully, I decided to keep the link there. My reason for this are:

<ol><li>Putting other issues aside, the article (or dialog) is still an excellent resource for explaining REST to everyone, regardless of skills or knowledges</li>
<li>Just please, please don't mind the title, I believe that the original writer doesn't mean any harm. Think about it, if you just replace the offending part with "for dummies" or "in layman term" this will be a non-issue actually. It's just his way of being witty (he is a programmer, see my explanation above).</li></ol></small>
</pre>

If you find that the article is TLDR; you can read my summarized understanding here:

<pre style="white-space: pre-wrap;">
<small>REST is an architectural concept created for <strong>distributed system</strong>, which consist of <strong>client</strong> and <strong>server</strong>. Both client and server work on an object called <strong>resource</strong>. Client might want to do something with the resource by calling <strong>request</strong> and server will process the request and return an appropriate<strong> response</strong>. 

Request and response will contain representation of the resource <strong>state</strong>, for request it will be the <strong>"intended"</strong> state and for response it will be the <strong>"current"</strong> or the <strong>"latest"</strong> state (after the request has been processed, whether success or failure). Request have several kind of <strong>verbs</strong> that it will use to communicate the client intention towards the resources to the server, which are: GET, POST, PUT and DELETE.

REST behave this way due to the characteristics and requirements of a distributed system. Some of it are:

1. Various kind of entity may participate, therefore <strong>compability</strong> must be preserved.
2. <strong>Connection-less</strong>, each entity may not maintain connectivity at all the time.
3. System must be <strong>scalable</strong>. It can be throttle up and down as needed.</small>
</pre>

## $resource and REST

Rails fully embraces REST from version 1.2 onwards (previously they use SOAP). DHH post about the transition in the rails blog, [here](https://weblog.rubyonrails.org/2007/1/18/rails-1-2-rest-admiration-http-lovefest-and-utf-8-celebrations/). Therefore if we adhere to Rails best practice when creating the controllers and routes (perhaps by following the [official guide](https://guides.rubyonrails.org/routing.html)), our resources will be RESTful. Thus no changes are needed in the Rails side. 

Let's take a look at all actions from our Rails `TasksController` and its corresponding routes below (standard CRUD actions).

<table>
  <tr>
    <td><strong>HTTP verb</strong></td>
    <td><strong>Path</strong></td>
    <td><strong>Action</strong></td>
    <td><strong>Used for</strong></td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/tasks</td>
    <td>index</td>
    <td>Display a list of all tasks</td>
  </tr>
  <tr>
    <td>POST</td>
    <td>/tasks</td>
    <td>create</td>
    <td>Create a new task</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/tasks/:id</td>
    <td>edit</td>
    <td>Return an existing task for editing</td>
  </tr>
  <tr>
    <td>PUT</td>
    <td>/tasks/:id</td>
    <td>update</td>
    <td>Update an existing task</td>
  </tr>
  <tr>
    <td>DELETE</td>
    <td>/tasks/:id</td>
    <td>destroy</td>
    <td>Delete an existing task</td>
  </tr>
</table>

At the Angular side, we already make a call to each of the above resources from the `TasksController`. Let's review the code below (some comments are omitted for brevity):

```coffee
# app/assets/javascripts/app/controllers/tasks-controller.js.coffee

app.controller "TasksController", ($scope, $http, $location, $state, $stateParams) ->
  $scope.tasks = {}
  $scope.task = {}

  if $state.current.name == 'tasks'
    $http.get("/api/tasks"

    # success
    ).then ((response) ->
      $scope.tasks = response.data
    
    # failure
    ), (error) ->

  if $state.current.name == 'edit'
    $http.get("/api/tasks/#{$stateParams['id']}"

    # success
    ).then ((response) ->
      $scope.task = response.data

    # failure
    ), (error) ->

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

  $scope.destroy = (id) ->
    $http.delete("/api/tasks/#{id}"

    # success
    ).then ((response) ->
      $http.get("/api/tasks").then ((response) ->
        $scope.tasks = response.data
      ), (error) ->

    # failure
    ), (error) ->
```

Each part of the code is actually an "action" that calls the corresponding Rails API. You may notice that each action calls the corresponding API using methods from Angular `$http` Service. Each utilized `$http` methods are directly mapped into a REST HTTP verbs, which are: `get`, `post`, `put` and `delete`.

Angular actually have a better way for handling calls to RESTful API rather than 'manually' through the usage of `$http`, which is using `$resource`. Below is the definition of `$resource` as explained by Angular official documentation.

<pre style="white-space: pre-wrap;">
A <a href="https://en.wikipedia.org/wiki/Factory_method_pattern">factory</a> which creates a resource object that lets you interact with RESTful server-side data sources.

The returned resource object has action methods which provide high-level behaviors without the need to interact with the low level $http service.
</pre>

Thus the utilization of `$resource` will provide benefits for our code in terms of increased simplicity and reduce errors. Next section will explain about utilizing `$resource` in our code.

## Using $resource in our previously created application

As usual you can find source code for example application on my repo, [here](https://github.com/giosakti/rails_angular_integration_example). I've tagged each part of the posts there, so you can easily switch to any part before this.

I've introduced some changes other than angular-resource-related, mainly for optimization. Summarized explanations for my non-related changes are:

- **<a href="https://github.com/giosakti/rails_angular_integration_example/commit/7a1767aa48bb4c8b0974ebf48d018e81a7c0cd7d">7a1767a</a> - jquery isn't mandatory**  
I remove jquery dependency (for now) to reduce load time and replace selector in `csrf.js.coffee` with vanilla js-equivalent.

- **<a href="https://github.com/giosakti/rails_angular_integration_example/commit/744c4476bb35bc109990b5c097e2102aa2ad4642">744c447</a> - remove unneeded test folder (probably will use spec for test)**  
I will likely use spec later for test, thus I remove this rails-generated test folder.

- **<a href="https://github.com/giosakti/rails_angular_integration_example/commit/c52648cd178ff91e50ee9e000fe5c232e8f323b1">c52648c</a> - rearrange and update angular dependency**  
Just update angular libraries and rearrange some stuffs.

- **<a href="https://github.com/giosakti/rails_angular_integration_example/commit/2d81e6187e8e9dc384bdd99dc8975172839c03a0">2d81e61</a> - put missing 'edit' action from Rails TasksController API**  
In the example from previous part, I forgot to put 'edit' action in the Rails API and call the 'show' action instead in the Angular side for editing. This commit put the missing 'edit' action.

**Include AngularResource in our application**

Now, we can discuss about steps that are required to utilize Angular Resource. First you have to 'install' (or include) the necessary Angular Resource library in your application. You can download the angular-resource library from their [offical repository](https://code.angularjs.org). As I used version 1.0.6 of angular, I downloaded the angular-resource [here](https://code.angularjs.org/1.0.6/angular-resource.js). After putting `angular-resource.js` file in `vendor/assets/javascripts/angular`, I require it at the `application.js` file.

```javascript
// app/assets/javascripts/application.js
...
// AngularJS
//= require angular/angular
//= require angular/angular-resource
//= require angular/angular-ui-states 
...
```

I also activate it in the `main.js.coffee` file.

```coffee
# app/assets/javascripts/app/main.js.coffee

# Create 'app' angular application (module)
@app = angular.module("app", [
  # ngResource
  "ngResource",

  # ui-router
  "ui.compat"
])
```

**Create our first resource: TaskResource**

Next we can create the resource itself by using the `$resource` factory. Creating it is pretty easy and the syntax is already covered in the Angular official documentation.

`$resource(url[, paramDefaults][, actions]);`

You can put the url for the resources, default parameters (to communicate with Rails you may want to specify `id` parameter here) and additional actions as necessary. By default angular will create 5 actions (actually 4, 2 of them are the same), which are: 

```
{ 
  'get':    {method:'GET'},
  'save':   {method:'POST'},
  'query':  {method:'GET', isArray:true},
  'remove': {method:'DELETE'},
  'delete': {method:'DELETE'} 
};
```

Remove and delete are actually the same method, but to ensure compability with IE you should use `remove` as `delete` is a reserved word (taken from [here](https://stackoverflow.com/questions/15706560/angular-resource-what-is-the-difference-between-delete-and-remove)). `get` is Rails equivalent for `show` action, `save` is `create`, `query` is `index` and `remove` (or `delete`) is `destroy`. The only missing action is angular-equivalent of Rails `update`, so we must add additional actions for that purpose. 

Our `TaskResource` will looks like this:

```coffee
# app/assets/javascripts/app/resources/task.js.coffee

app.factory "Task", ($resource, apiPrefix) ->
  $resource( apiPrefix + "/tasks/:id", 
    id: "@id"
  ,
    update:
      method: "PUT"
  )
```

We can also specify configurations for additional actions, such as method (as we do above), default parameters and `isArray`. `isArray` is useful if we want Angular to know that the returned resource is Array (for example, index action). 

Because I put `task.js.coffee` at a new directory:`app/assets/javascripts/app/resources/`, we must include it in the `application.js` file.

```javascript
// app/assets/javascripts/application.js
...
// app-specific js files
//= require app/main
//= require_tree ./app/controllers
//= require_tree ./app/resources
//= require_tree ./config
...
```

**Using TaskResource in our controller**

There are two ways to utilize `$resource` generated class. First is to call the methods directly from the generated class, which can be classified into calling `GET` actions or non-`GET` actions. For non-`GET` actions such as `POST` or `PUT` you also have to specify `postData` as a parameter. You can see the examples of calling non-`GET` actions below (this is from our actual TasksController).

```coffee
# app/assets/javascripts/app/controllers/tasks-controller.js.coffee

$scope.create = ->
  Task.save(
    {}
  , task:
      title: $scope.task.title
      description: $scope.task.description

    # Success
  , (response) ->
    $location.path "/tasks"

    # Error
  , (response) ->
  )
```

See that there are four parameters that we have to put: the first one is parameters to be sent (it's empty as create doesn't usually need any extra parameters), the second is `postData`, the third is function for handling success API response and the last one is function for handling error API response. If we call `GET` actions, such as `GET` or `DELETE`, we only have to omit `postData` parameter, the rest is the same.

The other way to utilize `$resource` generated class are by calling the methods from instance that was generated by the resource class. To do that first we have to retrieve the instance using the generated class. After that we can call `$save`, `$remove` and `$delete` directly to the object (all are non-`GET` actions, there is no `GET` actions for instance actions). You can see the example below:

```coffee
# Example of invoking actions from instance

task = User.get(
  id: 1

  # Success
, (response) ->
  task.title = 'change the title'
  task.$save()

  # Error
, (response) ->
)
```

As there are no `GET` actions for instance actions, you must be wondering why we don't specify `postData` at the snippet above. It is because the object is already `postData` itself, thus we don't have to put `postData` anymore in this case.

Now, knowing all the steps in utilizing `TaskResource` we can modify our code as necessary. [This](https://github.com/giosakti/rails_angular_integration_example/commit/143f60b508aa52de3361ffbbe05f470994729dd1) is what the final result will looks like.

## Conclusion

There you go, calling RESTful API conveniently. Next part will be about Angular Service and after that I will discuss about authentication, which is a bit more complicated.

**[Checkout the source here](https://github.com/giosakti/rails_angular_integration_example)**

and oh, pardon my lateness :)
