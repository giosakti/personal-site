---
title: "Interesting Tidbits about Reddot Ruby Conference (rdrc) 2013 - Day One"
authors: ["gio"]
date: 2013-07-06T14:19:00+07:00
tags:
- conference
- ruby
---

It's been quite a month for me, after moving my office from Bandung to Jakarta, I just got married last week and now I'm sitting here in the middle of forest, enjoying my honeymoon :) However I have apologize to make for all of you whom read my blog (none), for making you wait..

So, around three weeks ago I went to the Reddot Ruby Conference (RDRC) in Singapore. There were tons of interesting talks there that I want to share. Some of the speakers are well-known, such as Aaron Patterson [@tenderlove](https://twitter.com/tenderlove‎), Jose Valim ([@josevalim](https://twitter.com/josevalim)), Jim Weirich ([@jimweirich](https://twitter.com/jimweirich)) and Steve Klabnik ([@steveklabnik](https://twitter.com/steveklabnik)). There was also talk by Akira Matsuda ([@amatsuda](https://twitter.com/amatsuda‎)), one of Ruby core contributor regarding Ruby 2.0. 

In this post I will share some of the interesting tidbits plus my commentary regarding each subject.

(You can also check rdrc official website [here](https://www.reddotrubyconf.com/))

## Day One

#### Aaron Patterson - Polymorphism, Refactoring &amp; Caching

RDRC 2013 kicks-off with Aaron on stage tackling three topics: polymorphism, refactoring and caching. Aaron is part of ruby core team and rails core team. The topics are very interesting albeit quite difficult to swallow in the morning. Aaron himself also said beforehand that he is sorry to greet our morning with "extremely technical" topics (and lots of codes!). However Aaron light-hearted personality has won us over and keep us in full attention during his presentation (see: [Americas next top engineer](https://speakerdeck.com/tenderlove/americas-next-top-engineer)).

**Polymorphism**

First topic is about polymorphism. Yes, polymorphism. Perhaps everyone whom use Rails has known about it already. However Aaron does not talk about how to use polymorphism, rather he talks about fundamental of polymorphism. The other two topics are also the same, it was more about concepts rather than practical "HOW-TOs", something that I think is more important if you are striving to be a programmer who wants to "create" rather than just "use".

There are two reasons on why polymorphism was created. The first is to hide complexity and the other is for *decoupling*, so that logic can be removed from the caller. Separation of concern is indeed one of important topic for developers. Good development framework with good **separation of concern** gives way to better **job delegation** and **accountability**. 

```ruby
# Without polymorphism

def drive(car)
  if car.is_a?(Automatic)
    car.put_in_drive
    car.gas
  else
    car.press_clutch
    car.put_in_gear
    car.release_clutch
    car.gas
  end
end

# With polymorphism

class Automatic
  def go
    put_in_drive
    gas
  end
end

class Manual
  def go
    press_clutch
    put_in_gear
    release_clutch
    gas
  end
end

def drive(car)
  car.go
end
```

**Refactoring**

The second topic was about refactoring. Aaron suggested that we use graph to represent our code for better analysis during refactoring.

<!-- <img src="http://mightygio.com/wp-content/uploads/2013/07/parse_tree.png" alt=""> -->
<!-- <img src="http://mightygio.com/wp-content/uploads/2013/07/dependency_tree.png" alt=""> -->

Aaron actually shows us how he refactor `set_callback` method on `ActiveSupport::Callbacks` as an example. I'm not gonna discuss in detail on how Aaron refactor those particular piece of code, but I'm gonna just extract the important bits of Aaron's talk. However if you're still interested on how he refactor the particular piece of code you can check his slide <a href="https://speakerdeck.com/tenderlove/reddotrubyconf">here</a>.

After creating the tree, he suggested that we should always start by refactor the leaf node first then moves up from there. For each leaf node, two refactoring strategy that he propose is: **polymorphism** and **cache invariant** (there are actually TONS of ways or strategies for refactoring, this is just the two that he suggested in his talk). We have discussed polymorphism earlier, but Cache invariant is new terminology (well, at least new for me :) ). Cache invariant is a technique where we cache information that is very rarely changed so that our application can avoid accessing the database or doing repetitive process.

**Caching**

The last topic is about caching. As we use ActiveRecord as an example, Aaron first give us an explanation on how ActiveRecord query works. Where and nested where's is evaluated by Ruby as a LinkedList. The process on how ActiveRecord does its jobs when evaluating query are something like this:

1. Our code is converted to `ActiveRecord::Relation`
2. `ActiveRecord::Relation` then converted to SQL Abstract Syntax Tree (AST)
3. Lastly, SQL AST is converted to SQL statement and then got executed

Everything is generated on-the-fly, therefore Aaron thinks that it's not very efficient because every generated SQL statements are often very similar with only small part of it changed. Therefore Rails 3.2 introduced something called "bind params". Bind params is a technique where bind parameters are the only thing that changes and generated on-the-fly thus making the process more efficient and ultimately gives speed improvement for our application. In this case, query plan and parsed SQL is cached and the SQL statements are not entirely generated on-the-fly anymore. 

*You can check Aaron's slide [here](https://speakerdeck.com/tenderlove/reddotrubyconf)*

#### Luismi Cavalle - Keep your ActiveRecord Models Manageable The Rails Way

Aaron talks are very interesting, but this topic that was brought by Luismi Cavelle are more inline with what my current interest lie. Luismi is programmer from **BeBanjo**, a company based on Madrid which have products related to video on demand, metadata and sequence.

My biggest concern for using Rails is now about its capability in handling big project. What I mean by big is when the project has complex requirements and business process (I'm not currently talking about scalability). Some of my past projects have Rails application with tons of models and controllers. The models, especially, have bad traits that I hate: **crowded**, **complex** and **not very readable** especially for our developers whom joined the project late.

**Fat Model** was the first topic that was touched by Luismi. It was exactly the topic that I want to explore more. It seems that he has the same concern with me, that is a 'Fat Model', an active record model that does not adhere to [SRP (Single Responsibility Principle)](https://en.wikipedia.org/wiki/Single_responsibility_principle), have low cohesion, tight coupling with related things placed too far away thus making it hard to read (this will most likely happens if you have a model that have > 200 lines of codes).

**ActiveSupport::Concern**

There are several ways that can be used for handling Fat Model. One of the method that was introduced by Luismi is `ActiveSupport::Concern`. In the simplest explanation that I can give, `ActiveSupport::Concern` is a [mixin](https://en.wikipedia.org/wiki/Mixin). Just like a mixin in compass css framework. They are **classes** that bind together **common purposes**, which is not a part of model's nature, is cross-cutting and usually tightly related to a domain concept. These classes can be reused by our models by including them. Examples of an `ActiveSupport::Concern` is taggable, searchable, movable, visible, trashable. 

```ruby
module Taggable
  extend ActiveSupport::Concern
  
  included do
    has_many :taggings
    has_many :tags, through: :taggings

    scope :tagged_with, ->(tag_name) do
      joins(:tags).where(tags: {name: tag_name})
    end
  end

  def tag_list
    tags.pluck(:name).join(', ')  
  end

  def tag_list=(tag_list)
    assign_tag_list tag_list
  end

  private

  def assign_tag_list(tag_list)
    tag_names = tag_list.split(',').map(&:strip).uniq 

    self.tags = tag.names.map do |tag_name|
      Tag.where(name: tag_name).first_or_initialize 
    end
  end
end
```

```ruby
class Article < ActiveRecord::Base
  include Taggable
  ...
```

Some advantages on using `ActiveSupport::Concern` are they help keep cohesion high and ensuring DRYness of your models. However they have several disadvantages or 'concern' (ehe.. another pun :) ). They don't actually fix problems, instead they just move it and using ActiveSupport::Concern will increase your methods and responsibilities / burden. However if your project is complex I think using `ActiveSupport::Concern` is well worth it. By the way, you should put your concerns in `app/controllers/concerns` and `app/models/concerns`

**Non-persisted Model**

And then there is non-persisted model. Models in our rails projects are usually tightly related to a database object (persisted). However, there maybe exist some behaviors, which needed to be placed within its own model, but do not need to be persisted into the database. This one is perhaps similar with Service Layer - [Facade Pattern](https://en.wikipedia.org/wiki/Facade_pattern), just like Service in our Angular application. Non-persisted model is a service, an object that maybe called by another models / objects, which purpose is to give abstraction to a group of methods. Therefore, persistence is not needed.

```ruby

# Example of a service object

class Signup
  extend ActiveModel::Naming
  include ActiveModel::Conversion
  include ActiveModel::Validations

  attr_accessor :name
  attr_accessor :company_name
  attr_accessor :email

  validates :email, presence: true
  # ... more validations ...

  # Virtual models are never themselves persisted
  def persisted?
    false
  end

  def save
    if valid?
      persist!
      true
    else
      false
    end
  end

  private

  def persist!
    @company = Company.create!(name: company_name)
    @user = @company.users.create!(name: name, email: email)
  end
end
</pre>
```

```ruby
class SignupsController < ApplicationController
  def create
    @signup = Signup.new(params[:signup])

    if @signup.save
      redirect_to dashboard_path
    else
      render "new"
    end
  end
end
```

**Controllers.. Controllers for handling Fat Model??**

People often overlook controllers original purpose when creating Rails application. Too often that we put too much focus in "thin controller, fat model" mantra to end up with too fat of a model and too thin of a controller. [@dhh](https://twitter.com/dhh) himself actually said that it's ok to use controllers for orchestration. 

So for example, sending email? delegate to external services? it's actually ok to put the code in controllers for those purposes. And then callbacks actually should be avoided but rather be put into the controller. One rule the we must keep in mind is that we must extract code from controller when a logic is used in multiple places.

```ruby

# This is actually O.K

class CommentsController < ApplicationController
  # ...

  def create
    @comment = @post.comments.create!(post_params)

    Notifications.new_comment(@comment).deliver

    TwitterPoster.new(current_user, @comment.body).post
    FacebookPoster.new(current_user, @comment.body).post
  end
```

**Conclusion**

What we have discussed above are still all about options, it's all up to you regarding the implementation. However, I think that I will discuss about this topic further in a later post, because I find it very interesting and inline with my current work.

*You can check Luismi's slide [here](https://speakerdeck.com/cavalle/keep-your-activerecord-models-manageable-the-rails-way)*

#### Jim Weirich - Code Kata and Analysis

Jim is RDRC day one last presenter, it was also one of the most interactive presentation during the conference. Jim was doing a 'live coding' or 'code kata' as he called it. As I practiced kendo, I noticed the word **kata** is derived from japanese martial arts (Other than Kendo, I think kata also exist in Karate or probably other martial arts albeit with different name). Kata is a set of movements that must be practised regularly.

If I have to describe Code Kata by my own, I will describe it like this:

> Code Kata is honing your development workflow by solving an issue using code. You must be familiar with the chosen issue domain, but you must not known or at least pretend to not understand the solution beforehand. The purpose of code kata is to practice your worklow, so that you will instinctively adhere to it whenever working on another issue that you may encounter.

> Other advantage is that code kata makes sure that you are working bit-by-bit systematically so that it may be documented better and tackle big issue ellegantly rather than head-on.

The code kata practice by Jim in RDRC 2013 was using "creating a number to roman-numeral converter" as problem statement that needs to be tackled. Well explaining the process literally is a bit hard, you may want to check the [video](https://www.youtube.com/watch?v=983zk0eqYLY) (but this is in Boston, not during RDRC). However what Jim was doing was a very systematic and step-by-step development on stage when creating the tricky converter.

Jim was a very test-driven as developer, in the code kata example that was demo'ed by him. He has very strict rules in which we have to always, always **create tests first before writing our program**. On one time during the code kata practice that he forget to create test before writing the code, he reminded us not to. He also give tips to us that we should never, ever **refactor a failing test**.

> Code Kata "Rules"  
> Know: Where to Start  
> Know: Where to Continue  
> Know: The Solution to Drive the tests  
> Know: What to Skip  
> Know: To Recognize Duplication  
> Know: When to Leave in Duplication  
> Know: The Edge Cases  

You absolutely have to check [this](https://codekata.pragprog.com) if you're interested about code kata.

## Conclusion

Okay then, that is all for day one. See you in day two.
