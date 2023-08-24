---
title: "Active Record 101: A Beginner’s Guide to Structure, Form, and Nullify"
 
slug: active-record-tricks
 
tags: ruby, rails, active-record
 
domain: rjrobinson.hashnode.dev
 
subtitle: 
 
 
# SEO Title
# String | Optional
# Ex: Top 4 React UI Libraries for 2023
# - This is equivalent to SEO TITLE option present in draft settings
#   of a post in Hashnode editor.
# - You can also update a post to update this later.
seoTitle: "Understanding Active Record in Ruby on Rails: A Beginner's Guide to Callbacks, Associations, and Validations"
 
seoDescription: Explore the essentials of Active Record in Ruby on Rails with this beginner's guide. Dive into callbacks, associations, validations, scopes, and more. Learn how to manage database relationships with ease and understand the importance of method order to achieve the desired behavior.
 
disableComments: false
 
enableToc: true
 
saveAsDraft: false
 
 This blog is a beginner's guide to Active Record in Ruby on Rails, covering topics such as callbacks, associations, validations, scopes, class methods, and instance methods. It emphasizes the importance of understanding the order of methods and how they interact with each other for achieving the desired behavior.

```ruby
# Callbacks
# Associations
# Validations
# Scopes
# Class methods
# Instance methods
```

Active Record is like a magical unicorn that makes database management a breeze in Ruby on Rails. But like all unicorns, it can be a bit of a headbanger if you don’t understand how it works. That’s why I’m here to take you on a journey of why being mindful of how Ruby evaluates things is important.

```ruby
class Book < ApplicationRecord
  # Callbacks
  before_destroy :do_this  
   
  # Associations
  has_many :authors, dependent: :nullfiy
  
  # Validations
  validates :page_numbers
  
  # Scopes
  scope :reviewed, -> { where(reviewed: true) }
   
  def do_this
    "like, comment, subscribe" 
  end 
  
  # class methods
  def self.new_with_review
    # do magic things
  end
  
  # instance methods
  def read
    # do magic things
  end
end

```

I was working with a friend, and he presented an issue he had where he wanted authors to be nullified upon destroying the book. ( class and method names have been changed to protect the innocent ).

In the example, the #before_destroy callback is placed below the scopes, which can be like trying to fight crime in a tutu — it might work, but it’s not the best idea. In the testing, every time he called #destroy on the record, it did what it needed to; it nullified the association, but the behavior he was looking for was that he wanted the associated records to be “#touched” before the nullification/destroy. When we tried to debug and got inside the “#do_this” call back that we had defined, we could not pull the records we were expecting because, at this point, the nullification had already happened. Nullification happened before our customer #do_this was called.

As we went through his code, we could not figure out what was happening.

Callbacks! They’re like the superhero sidekicks of Active Record, always there to save the day when you need them. For example, the #before_destroy callback is like Batman’s butler, Alfred. It always ensures everything is in order before the record is destroyed. In the example code, the #before_destroy callback calls the #do_this method, which performs some action. But like Batman’s bat signal, the order of callbacks is critical.

Looking at the amazing documentation, we found that there is no #before_nullify call back. Or is there? I’ll admit — I had to recruit my own rubber duck for this one so special thanks to JM :)

```ruby
class Book < ApplicationRecord
   # Associations
  has_many :authors, dependent: :nullfiy
  
  # Validations
  validates :page_numbers
  
  # Scopes
  scope :reviewed, -> { where(reviewed: true) }
  
  # Original Location! <<<<>>>>>>
  # Callbacks
  before_destroy :do_this
  
  def do_this
    "like, comment, subscribe" 
  end 
  
  # class methods
  def self.new_with_review
    # do magic things
  end
  
  # instance methods
  def read
    # do magic things
  end
end
```

Looking at this `has_many :authors, dependent: :nullfiy` we see that the `dependent: :nullify` is present, and thus called before our custom #before_destroy callback kicks off. This is where, sometimes, the order in which callbacks are declared in your model matter.

The solution? Move the #before_destory callback to the top of the file. This way, it processes it sequentially, 1st the #do_this, and second the `dependent: :nullify` is called nulling out the IDs as expected.

This isn't to say that the above structure should be followed like dogma, but rather to be aware that sometimes…. order of methods matters.

Next up, we have associations. They’re like the family and friends of Active Record. They help you create relationships between records. For example, the has_many association is like having a big family. It creates a one-to-many relationship between two models. In the example code, the has_many :authors, dependent: :nullify association is like having a big family, but one of the kids keeps breaking things so you have to send them to their room (nullify). But like family dynamics, the order of methods can affect the behavior of the association.

Validations are like the bouncers at a club. They make sure that only the cool records get in. For example, in the example code, the `validates :page_numbers` validation is like the bouncer checking ID’s at the door. It ensures that the page_numbers attribute of a Book record is valid before it can be saved.

Scopes are like the filters on Instagram. They make sure that only the good stuff is shown. For example, the `scope :reviewed, -> { where(reviewed: true) }` scope is like the “show me only the cute dogs” filter. It ensures that only Book records with a reviewed attribute set to true are returned.

Active Record also provides support for class methods and instance methods. Class methods are like the cool kids at school. They’re defined on the class level and can be called on the class itself—for example, the def self.new_with_review class method is like the cool kid who always knows the latest fashion trends. It creates a new Book record with a reviewed attribute set to true. Instance methods, on the other hand, are like the nerdy kids at school. They’re defined on the instance level and can be called on an instance of a class. For example, the def read instance method is like the nerdy kid who always has their nose in a book. It performs some action on a specific Book record.

In conclusion, Active Record is like a magical unicorn that makes database management a breeze in Ruby on Rails, but like all unicorns, it can be a bit of a headbanger if you don’t understand how it works. Understanding the order of methods and how they interact with each other is crucial to achieving the desired behavior. So, remember that the order of methods matters and have fun with your code.
---
