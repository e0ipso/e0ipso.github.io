---
title: "Drupal's Bundle Classes Empower Better Code"
category: drupal
categories:
  - web-development
  - drupal
tags:
  - Drupal Development
image: /assets/images/2021/fruit.jpg
canonical_path: https://www.lullabot.com/articles/drupals-bundle-classes-empower-better-code
---
A highlight of the new features enabled by the recent commit to Drupal core and a comparison with the contributed Typed Entity module.

<!-- more -->
_This is a re-post of the article on the [Lullabot blog](https://www.lullabot.com/articles/drupals-bundle-classes-empower-better-code)._

Custom bundle classes have [landed in core](https://www.drupal.org/project/drupal/issues/2570593), which creates the possibility of improving the maintainability and quality of our PHP code. This article will explore how bundle classes can be leveraged and how this might lead to better code. This new feature will be available in Drupal core starting in 9.3.0. If you cannot wait for your site to upgrade to 9.3, you can use the [final patch](https://www.drupal.org/files/issues/2021-10-20/2570593-209.patch) in the issue queue for your site until then.

First, what makes "better" code? Better code should:

*   Be more declarative. This means that the intentions of what the code is doing should be more clear.
*   Be more testable. Testable code has explicit declaration of its dependencies and clear inputs and their outputs.

With that in mind, let's explore what custom bundle classes are. According to the [change record](https://www.drupal.org/node/3191609), bundle classes are classes that encapsulate business logic around entities.

> Entity bundles are essentially business objects, and now they can declare their own class, encapsulating the required business logic.

The change record offers a good amount of situations where this will be useful. These are organized in the following section on that page:

*   _Moving from template preprocess to get\*() methods_. The Twig integration in Drupal has always let you call methods like getFoo inside of the templates. In the past, we could not write our own getters for classes like Node or Term, so that feature was underused. Now that we can write these classes, we can drop **some** code from preprocess functions, calling getFoo directly in the template.
*   _Sharing code_. Following from the previous point, once we take code from procedural preprocess functions to a class (directly or via a trait, base class, etc.), we can reuse that code more easily in other places.
*   _Writing automated tests_. The more code we put in classes instead of procedural functions, the easier to test that code becomes. We will be able to leverage [kernel testing](https://www.drupal.org/docs/automated-testing/types-of-tests) (but not unit tests) for our custom logic encapsulated into these bundle classes.

Using Bundle Classes
--------------------

The [change record](https://www.drupal.org/node/3191609) covers how to make Drupal return the custom classes after loading an entity. Please refer to that. This section will reason about a hypothetical use case of this new feature.

Imagine you are building a website for a public library. It is reasonable to model books as a content type. Your team has decided that a node for books is a good approach since you will need a page for each book, along with moderation, translation, etc. Then, you proceed to create a bundle class called Book:

```php
    // Book.php
    class Book extends Node implements BookInterface {}
```

So far, this class does very little, but we can already see some benefits. When writing code that uses nodes, you can know that "a node is a book" from a static perspective using `$node instanceof BookInterface`, as opposed to the previous runtime perspective `$node->bundle() === 'book'`. This is important because your IDE can leverage this information and autocomplete for `BookInterface` but not `SectionInterface`. Not only will IDEs benefit from this, but also other tools like Psalm and PHPStan. You opened up so many possibilities with that little code!

Now it's time for your team to turn the attention to the `BookInterface`. You need that interface to encode the logic of your project, the business logic (remember, we said _bundles are essentially business objects_). You decide that books need to implement `AccessibleInterface` because they asked your team to restrict access to books based on their parental rating and the user's age.

```php
    // BookInterface.php
    interface BookInterface extends AccessibleInterface {}
```

This small piece of code, again, opens a ton of possibilities. You can now implement `hook_entity_access` and call the `->access(...)` method in the `$entity` if `$node instanceof AccessibleInterface`. This has two benefits. One, your access logic is contained within the Book class, along with the rest of the book methods (and can be tested). Two, if another bundle needs access checks, you don't need to update your `hook_entity_access` implementation, your team can just add `AccessibleInterface` to the bundle's interface.

Let's keep (fake-)coding. These two small changes have delivered so much! Next, you want to ensure labels display as the stakeholder requires, using the pattern "Title - Editorial (Year)." However, on cards, the label should omit the word "Editorial."

```php
    // Book.php
      public function getTitle(string $view_mode = ''): TranslatableMarkup {
        if ($view_mode === 'card') {
          return $this->t('@title (@year)', [
            '@title' => $this->get('title')->value,
            '@year' => $this->yearFromTimestamp($this->get('field_release_date')->value),
          ]);
        }
        return $this->t('@title - @editorial (@year)', [
          '@title' => $this->get('title')->value,
          '@editorial' => $this->get('field_editorial')->entity->label(),
          '@year' => $this->yearFromTimestamp($this->get('field_release_date')->value),
        ]);
      }
```

In the Twig template, we can call this to invoke the new method:

```twig
    {# node--book-twig.html #}
    <h2>{ { node.getTitle() }}</h2>
```

Your `node--book.twig.html` template can use this method to print the book title as defined by the `Book` class. You just created a small and testable method for rendering the book title. More importantly, you can do this for other fields and keep them organized. In the past, this could have been a deeply nested `if` statement in a preprocess function living alongside all the other field preprocessing for all the content types. This may not seem big from a 'preprocess tangle just works' perspective. However, when you need to apply that same logic anywhere else (for instance, some Views code), untangling the preprocess code to extract that logic to a custom method you can reuse will be cumbersome and likely to result in code duplication.

Additionally, if developers want to see if they could reuse something, this new approach allows them to look in a single place. Without bundle classes, the logic could be in your theme, in a custom module `.module`, abstracted in a service, or anywhere!

As you can see, using bundle classes leads to many benefits, even with our tiny examples. Imagine what it can do for your complex project.

Where Bundle Classes Fall Short
-------------------------------

Bundle classes are a huge step forward. Unfortunately, they fall short for many use cases. They may be enough for your needs. As shown above, just by using bundle classes, you are already improving your codebase.

But in the future, you may feel you need "something" more. When that happens, you may want to look at the [Typed Entity](https://www.drupal.org/project/typed_entity) module.

The first version of Typed Entity for Drupal 7 was released in February 2015. Back then, the classes extended the entities ([sort](https://git.drupalcode.org/project/typed_entity/-/blob/7.x-1.x/src/TypedEntity/TypedEntity.php#L215-232) [of](https://git.drupalcode.org/project/typed_entity/-/blob/7.x-1.x/src/TypedEntity/TypedEntity.php#L215-232), entity objects were not a thing then). Now, the module utilizes [composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance).

If you want to dive deeper into this topic, take a look at [Maintainable Code in Drupal: Wrapped Entities](https://www.lullabot.com/articles/maintainable-code-drupal-wrapped-entities) and [Write Better Code with Typed Entity](https://www.lullabot.com/articles/write-better-code-typed-entity). If you learn better with video, [this presentation](https://www.youtube.com/watch?v=TQMrfH9oDX4) is available.

For our immediate purposes, bundle classes are compared with the Typed Entity approach in [minute 8:23 of the video presentation](https://youtu.be/TQMrfH9oDX4?t=503). [This issue in Typed Entity's issue](https://www.drupal.org/project/typed_entity/issues/3245990) queue prompted that exploration. Let's explore some more!

<img src="/assets/images/bundles-1.png" style="background-color: white; padding: 15px" alt="Comparison" />

### **Bundles are not Atomic**

We have established that _entity bundles are essentially business objects_. However, they are almost always _multi__\-__purpose_ business objects. Imagine that you add a `field_book_type` field to allow editors to specify whether a book is _paper_ or _audio book_, and then you add another a `field_book_popularity` to select _hidden gem_, _popular_, _best seller_. It is reasonable to think there will be specific logic for audiobooks or bestsellers, just like we had specific logic for articles before, but we only had the `Node` class.

How will that look in code when using bundle classes?

```php
    // Book.php
      public function getLink(): Link {
        /* ... */
        if ($this->getType() === 'audio book') {
          // Link to the MP3 player page.
          /* ... */
        }
        else {
          // Build the link to the PDF asset and create the Link object.
        }
      }

      public function getTitle(string $view_mode = ''): TranslatableMarkup {
        /* ... */
        if ($this->getPopularity() === 'best seller') {
          // Add the the "best seller badge".
          /* ...  */
        }
      }
```

At this point, you might be asking yourself:

1.  _How is this_ `_$this->getType() === 'audio book'_` _different from the_ `_$node->bundle() === 'book'_` _we were trying to avoid above?_
2.  You might follow that up with _We have improved greatly moving away from the big tangle of nested_ `_if_` _in the preprocess functions, but I have a smaller mess in my_ `_getXYZ_` _functions_.
3.  Then, you realize that _I still want those IDE and static analysis benefits!_

[Typed Entity](https://www.drupal.org/project/typed_entity) addresses this problem and lets you create book "variants" based on arbitrary conditions. In this case, we could create a the `AudioBook` and `BestSellerBook` variants. Then we will benefit from everything bundle classes introduced inside of the bundles.

```php
    // AudioBook.php (extends Book.php)
      public function getLink(): Link {
        // Link to the MP3 player page.
        return /* ... */
      }

      // Method specific for audio books.
      public getTranscriptions(): string {
        /* ... */
      }
    // Book.php
      public function getLink(): Link {
        // Build the link to the PDF asset and create the Link object.
        return /* ... */
      }
```

As you can see, Typed Entity will empower you to have **dedicated classes to your business objects**, regardless of whether they are entity bundles or not.

### **Mixing Framework Logic and Business Logic**

From the Wikipedia page on [Business Logic](https://en.wikipedia.org/wiki/Business_logic):

> In computer software, business logic or domain logic is the part of the program that encodes the real-world business rules that determine how data can be created, stored, and changed. It is contrasted with the remainder of the software that might be concerned with lower-level details of managing a database or displaying the user interface, system infrastructure, or generally connecting various parts of the program.

In Drupal words, entities and bundles are Drupal concepts. Books, audiobooks, buildings, etc., are concepts from _your project_. Ensuring they are separate will improve code quality, as reasoned by [SOLID](https://simple.wikipedia.org/wiki/SOLID_(object-oriented_design)) design principles, particularly the single responsibility principle.

When we make Book a bundle class that extends from Node, we are dragging all the framework logic necessary to make nodes function (database, render, routing, ...) into our business logic.

Instead, the Book class should **use** the node (database, render, routing, etc.) and not **be** a node. With that in mind, when you use Typed Entity, your Book class will have a $node property attached to use instead of extending the Node class.

When your Book has a node in it, you can write Unit tests for it **mocking the dependency** on the $node. You cannot mock the node to test the book with bundle classes because the book is the node.

The major drawback is that you will need to "wrap" your entities in the integration points between framework and business logic. This is why in Typed Entity, they are called _wrapped entities_. This `wrap($entity)` is likely to be in hooks, events, services, etc. This is simple to do, but some feel it is tedious. Bundle classes will not suffer from this because they **are** the entities.

This is what wrapping an entity looks like:

```php
    function mymodules_entity_presave(EntityInterface $entity) {
      $wrapped_entity = typed_entity_repository_manager()->wrap($entity);
      // ...
    }
```

Within your business classes, you will not need to wrap your entities manually; you can use the `wrapReference()` and `wrapReferences()` methods. This inconvenience happens only where business logic interacts with framework logic.

```php
    // AudioBook.php
      public function getNarrator(): Person {
        // Do NOT use this:
        // return typed_entity_repository_manager()->wrap($this->getEntity()->get(static::FIELD_NAME_NARRATOR)->entity);
        //
        // Use this instead:
        return $this->wrapReference(static::FIELD_NAME_NARRATOR);
      }
```

Notice how `getNarrator` returns a business object `Person`, not an entity.

### **Business Logic for Multiple Entities**

The last inconvenience with bundle classes is that they do not offer a solution for logic that applies to multiple items or no particular item. In our imaginary project, we might want to update the lending status to "unavailable" for all the books with no copies left in the building. With bundle classes, we will need to resort to [static methods](https://en.wikipedia.org/wiki/Method_%28computer_programming%29#Static_methods). However, static methods are just procedural functions with a namespace. They suffer from the same problems we want to solve.

To address this, Typed Entity uses typed repositories. Hence, we will have a BookRepository class that will be in charge of these types of operations:

```php
    // BookRepository.php
    class BookRepository extends TypedRepositoryBase {
      // To be run by a drush command every day at night.
      public function updateBooksFromLendRecords(array $isbns): void {
        $books = $this->loadFromMultipleIsbns($isbns);
        array_map(function (BookInterface $book) {
          $book->loan();
        }, $books);
      }
    }
```

With typed entity, we end up with a regular method we can test. We could easily mock `loan` on the `Book` class. On the other hand, the static functions in bundle classes are not testable. Moreover, if this logic needs to be used in a Drush command, it would call `Book::updateBooksFromLendRecords`, which you will not be able to mock. This makes the Drush command untestable as well.

With Typed Entity, the Drush command receives the book repository via dependency injection so that you can test as usual.

<figure>
  <img src="/assets/images/bundles-2.png" style="background-color: white; padding: 15px" alt="Diagram" />
  <figcaption>The book repository is in charge of operations on multiple books. Each object can be an AudioBook, a plain Book, etc.</figcaption>
</figure>

### **Other Useful Features**

In addition to the features above, Typed Entity also introduces the concept of [_renderers_](https://git.drupalcode.org/project/typed_entity/-/blob/4.x/src/Render/TypedEntityRendererInterface.php). Renderers are business objects for rendering an entity. They facilitate writing code for [`hook_preprocess`](https://api.drupal.org/api/drupal/core!lib!Drupal!Core!Render!theme.api.php/function/hook_preprocess), [`hook_entity_view_alter`](https://api.drupal.org/api/drupal/core!lib!Drupal!Core!Entity!entity.api.php/function/hook_entity_view_alter), and [`hook_entity_display_build_alter`](https://api.drupal.org/api/drupal/core!lib!Drupal!Core!Entity!entity.api.php/function/hook_entity_display_build_alter) in a more reusable and testable way. They also allow you to organize the business logic for rendering a particular entity. Renderers exist as a statement to say: _rendering entities into HTML is the most common goal for them to exist,_ _so_ _let's provide helpers for that custom code to be more manageable._

Typed Entity also comes with a sub-module for developers called Typed Entity UI. This tries to help with onboarding new developers into a project so that they can see the business objects for entities with a glance. Something similar for bundle classes would also be useful.

![UI](/assets/images/bundles-3.png)

In Conclusion
-------------

Bundle classes are a great step forward for Drupal development. They open the door to more maintainable code with little effort. Just one extra step that will make your projects more sustainable and reliable. It also makes projects more enjoyable to work with for developers.

Both Typed Entity and bundle classes are very useful. They will allow you to improve the quality and maintainability of your codebases to different degrees. However, they are just tools. They will not do the work for you. But they will help you leverage object-oriented programming design patterns, set internal conventions within your team for code organization, leverage automated testing of business logic where it makes sense, and much more.

For many reasons, bundle classes still fall short. There are still things that need to be improved. Because improvements to Drupal core are always complex, while contributed modules can be more nimble, Typed Entity will remain a place where these pain points can continue to be addressed.

<small>Photo by <a href="https://unsplash.com/@amyshamblen">Amy Shamblen</a> on <a href="https://unsplash.com/s/photos/fruit">Unsplash</a></small>
