---
layout: post
title: The Null Object pattern
summary: A technique for dealing with certain nil errors
date: 2015-06-07
published: true
categories: ruby
---

Every Ruby programmer eventually encounters the dreaded `NoMethodError` exception.

```rb
undefined method `first_name' for nil:NilClass
```

We learn to live with it and handle it in various nasty ways. Like using `try`:

```slim
tr
  td = Client.human_attribute_name(:vat_number)
  td = @client.try(:vat_number)
tr
  td = Client.human_attribute_name(:company_ownership_type_id)
  td = @client.ctry(:company_ownership_type, :name_bg)
tr
  td = Client.human_attribute_name(:company_branch_id)
  td = @client.ctry(:company_branch, :name_bg)
```

This code is bad because our `@client` should never actually be `nil`. We can improve it by using the
Null Object Pattern.

### An example

Imagine we have an application and have to print a greeting to the current user, something like
"Welcome, George". The first thing we try may be:

```rb
Welcome, <%= current_user.name %>
```

But then we log out and are greeted with an old friend:

```rb
NoMethodError: undefined method `name' for nil:NilClass
```

We quickly fix our code by adding a `nil` check and also decide we want to greet logged-out people
too:

```rb
Welcome, <%= current_user.nil? ? 'Guest' : current_user.name %>
```

This works as we originally intended and is good enough for production. Or is it? We start using
the same pattern in different places, our collegues also pick it up and a few months later we find
ourselves in with filthy mess that is really hard to refactor.

Our next problem comes with a change in the requirements - guest users are not to be greeted as
"Guest", but as "Stranger", because our boss thinks it's funnier. We start working on our task and
find that there are 78 occurances of the string "Guess" in our view code and we need to change each
one separately because we cannot mass replace such a common word as 'Guest'. We curse the gods and
get on with it, because this is life and life sucks.

Could this have been avoided? Yes, and the solution is really simple.

### The pattern

What we need is way to make `current_user` respond to the `name` method even if no user is
currently logged in. We start by introducing a new class:

```rb
class NullUser
  def name
    'Guest'
  end

  def logged_in?
    false
  end
end
```

The `NullUser` class should conform to the same public API as the `User` class, which requires
maintenance, but everything comes at a cost.

Next, we change our `current_user` method to return a new instance of `NullUser` when no user is
logged in:

```rb
def current_user
  current_user_session || NullUser.new
end
```

Now when our new requirements arrive, we only need to update the `name` method of the `NullUser`
class and need not to worry about breaking anything else. If we dislike the name of the
class, we may also use `Guest` or `GuestUser`, it doesn't matter.

### Drawbacks

The Null Object is not a silver bullet and not something you want to use for all your classes. It
should not be used to avoid all `nil` errors but as a way to provide default behaviour when you get
an unexpected `nil`. Sometimes `nil` is just `nil` and should be handled as such. You should also
have an extensive test suite that ensures that nowhere a `nil` may be assigned instead of your Null
Object instance, because that will break wreak havoc on your happy little world.

### Last words

The pattern really shines in the cases it's is suited for, such as the NullUser or representing the
last element of a list/tree. Apply it with care and it may improve your life a little.
