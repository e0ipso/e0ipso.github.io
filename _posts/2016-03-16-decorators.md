---
title: "Get a decorator for your Drupal home"
categories:
 - web-development
tags:
 - Decoupled Drupal
 - Drupal Development
canonical_path: https://www.lullabot.com/articles/get-a-decorator-for-your-drupal-home
---
_This is a re-post of the article I wrote for the [Lullabot blog](https://www.lullabot.com/articles/get-a-decorator-for-your-drupal-home)._

<h2>Design patterns</h2>

<a href="https://en.wikipedia.org/wiki/Software_design_pattern">Software design patterns</a>&nbsp;are general and <strong>reusable</strong>&nbsp;software design <strong>solutions</strong>&nbsp;to a defined problem. They were popularized in 1994 by the <em>“Gang of Four”</em>&nbsp;after their book <a href="http://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612">Design Patterns: Elements of Reusable Object-Oriented Software</a>.
<!-- more -->
These generic solutions are not snippets of code, ready to be dropped in your project, nor a library that can be imported and reused. Instead they are a templated solution to the common software challenges in your project. Design patterns can also be seen as <strong>best practises&nbsp;</strong>when encountering an identified problem.

With Drupal 8’s leap into modern PHP, design patterns are going to be more and more relevant to us. The change from (mostly) procedural code to (a vast majority of) object oriented code is going to take the Drupal community at large through a journey of <strong>adaptation to the new programming paradigm</strong>. Quoting the aforementioned <em>Design Patterns: Elements of Reusable Object-Oriented Software</em>:

<cite>[…] Yet experienced object-oriented designers do make good designs. Meanwhile new designers are overwhelmed by the options available and tend to fall back on non-object-oriented techniques they've used before. It takes a long time for novices to learn what good object-oriented design is all about. Experienced designers evidently know something inexperienced ones don't. What is it?</cite>

Even if you don’t know what they are, you have probably been using design patterns by appealing to <em>common sense</em>. When you learn what they are you’ll be thinking “Oh! So that thing that I have been doing for a while is called an adapter!”. Having a label and knowing the correct definition will help you <strong>communicate better</strong>.

<h2>The decorator</h2>

Although there are <a href="https://en.wikipedia.org/wiki/Software_design_pattern#Classification_and_list">many</a> design patterns&nbsp;you can learn, today I want to focus in one of my favorites: <a href="https://en.wikipedia.org/wiki/Decorator_pattern">the decorator</a>.

The decorator pattern allows you to do <strong>unobtrusive behavior inheritance</strong>. We can have many of the benefits of inheritance and polymorphism without creating a new branch in the class’ inheritance tree. That sounds like a mouth full of words, but this concept is especially interesting in the Drupal galaxy.

In Drupal, when you are writing your code, you cannot possibly know what other modules your code will run with. Imagine that you are developing a feature that enhances the <code>entity.manager</code>&nbsp;service, and you decide to swap core’s <code>entity.manager</code>&nbsp;service by your enhanced version. Now when Drupal uses the manager, it will execute your manager, which extends core’s manager and overrides some methods to add your improvements. The problem arises when there is another module that does the same thing. In that situation either you are replacing that module’s <em>spiced</em>&nbsp;entity manager or that module is replacing your improved alternative. There is no way to get both improvements at the same time.

This is the type of situations where the decorator pattern comes handy. You cannot have this new module inheriting from your manager, because <strong>you don’t know</strong> if all site builders want both modules enabled at the same time. Besides, there could be an unknown number of modules –even ones that are not written yet–&nbsp;that may create the conflict again and again.

<h2>Using the decorator</h2>

In order to create our decorator we’ll make use of the interface of the object we want to decorate. In this case it is <code>EntityManagerInterface</code>. The other key component is the object that we are decorating, let’s call it <strong>the subject</strong>. In this case our subject will be the existing object in the <code>entity.manager</code>&nbsp;service.

Take for instance a decorator that does some logging every time the <code>getStorage()</code>&nbsp;method is invoked, for debugging purposes. To create a decorator you need to create a pristine class that implements the interface, and receives the subject.

```
<code class="language-php">class DebuggableEntityManager implements EntityManagerInterface {

 protected $subject;

 public function __construct(EntityManagerInterface $subject) {

   $this-&gt;subject = $subject;

 }

}
```

The key concept for a decorator is that we are going to <strong>delegate </strong>all<strong> method calls</strong> to the subject, except the ones we want to override.

```php
class DebuggableEntityManager implements EntityManagerInterface {

 protected $subject;

 // ...

 public function getViewModeOptions($entity_type_id) {

   return $this-&gt;subject-&gt;getViewModeOptions($entity_type_id);

 }

 // …

 public function getStorage($entity_type) {

   // We want to add our custom code here and then call the “parent” method.

   $this-&gt;myDebugMethod($entity_type);

   // Now we use the subject to get the actual storage.

   return $this-&gt;subject-&gt;getStorage($entity_type);

 }

}
```

As you have probably guessed, the subject can be the default entity manager, that other module’s spiced entity manager, etc. In general you’ll take whatever the <code>entity.manager</code>&nbsp;service holds. You can use any object that implements the <code>EntityManagerInterface</code>.

Another nice feature is that you can decorate an already decorated object. That allows you to have multiple decorators adding different features without changing the inheritance. You can now have a decorator that adds logging to every method the entity manager executes, on top of a decorator that adds extra definitions when calling <code>getFieldDefinitions()</code>, on top of …

I like the <a href="https://en.wikipedia.org/wiki/Decorator_pattern#Second_example_.28coffee_making_scenario.29">coffee making example</a>&nbsp;in the decorator pattern entry in Wikipedia, even if it’s written in Java instead of PHP. It’s a simple example of the use of decorators and it reminds me of delicious coffee.

<h2>Benefits and downsides</h2>

One of the downsides of using the decorator pattern is that you can <strong>only act on public methods</strong>. Since you are –intentionally– not extending the class of the decorated object, you don’t have access to any private or protected properties and methods. There are a couple of situations similar to this one where the decorator pattern may not be the best match.

The business logic you want to override is contained in a protected method, and that method is reused in several places. In this case you would end up overriding all the public methods where that protected one is called.

You are overriding a public method that is executed in other public methods. In such scenario you would not want to delegate the execution to the subject, because in that delegation your overridden public method would be missed.

If you don’t find yourself in one of those situations, you’ll discover that the decorator has other <strong>additional benefits</strong>:

<ul>
	<li>It favors the <a href="https://en.wikipedia.org/wiki/Single_responsibility_principle">single responsibility principle</a>.</li>
	<li>It allows you to do the decoration during <a href="https://en.wikipedia.org/wiki/Run_time_(program_lifecycle_phase)">run-time</a>, whereas subclassing can only be done in <a href="https://en.wikipedia.org/wiki/Compile_time">compile time</a>.</li>
	<li>Since the decorator pattern is one of the commonly known design patterns, you will not have to thoroughly describe your implementation approach during the <a href="http://www.mountaingoatsoftware.com/agile/scrum/daily-scrum">daily scrum</a>. Instead you can be more precise and just say “I’m going to solve the problem using the decorator pattern. Tada!”.</li>
</ul>

<h2>Write better designs</h2>

Design patterns are a great way to solve many complex technical problems. They are a heavily tested and discussed topic with lots of examples and documentation in many programming languages. That does <strong>not</strong> imply that they are your <strong>new golden hammer</strong>, but a very solid source of inspiration.

In particular, the decorator pattern allows you to add features to an object at run-time while maintaining the object’s interface, thus making it compatible with the rest of the code without a single change.
