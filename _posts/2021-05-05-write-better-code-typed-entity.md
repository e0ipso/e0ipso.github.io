---
title: 'Write better code with Typed Entity'
category: drupal
categories:
  - web-development
  - drupal 
tags:
  - Drupal Development
image: /assets/images/2021/laptop.jpg
canonical_path: https://www.lullabot.com/articles/write-better-code-typed-entity
---
I proposed this session to DrupalCon, but it was not selected. I think that is good. I have had my fair share of stage
time in DrupalCons in the past, new contributors should take the lead. However, I still did the work of creating
the presentation, then recorded myself giving the talk.

<!-- more -->
_This is a re-post of the article on the [Lullabot blog](https://www.lullabot.com/articles/write-better-code-typed-entity)._

<div class="video-wrapper"><iframe sandbox="allow-same-origin allow-scripts allow-popups" src="https://video.mateuaguilo.com/videos/embed/6da46367-2ac7-40d1-b561-095b991f7dd2" frameborder="0" allowfullscreen></iframe></div>

<a href="https://mateuaguilo.com/assets/documents/write-better-code-with-typed-entity.pdf">Slides available here</a>.

Drupal projects can be challenging. You need to have a lot of framework-specific knowledge or Drupalisms. Content types, plugins, services, tagged services, hook implementations, service subscribers, and the list goes on. You need to know when to use one and not the other, and that differs from context to context.

It is this flexibility and complexity that allows us to build complex projects with complex needs. Because of this flexibility, it is easy to write code that is hard to maintain.

How do we avoid this? How do we better organize our code to manage this complexity?

## Framework logic vs. business logic

To start, we want to keep our framework logic separate from our business logic. What is the difference?

- **Framework logic** \- this is everything that comes in Drupal Core and Drupal contrib. It remains the same for every project.
- **Business logic** \- this is what is unique to every project—for example, the process for checking out a book from a library.

The goal is to easily demarcate where the framework logic ends and the business logic begins, and vice-versa. The better we can do this, the more maintainable our code will be. We will be able to reason better about the code and more easily write tests for the code.

## Containing complexity with Typed Entity

Complexity is a feature. We need to be able to translate complex business needs to code, and Drupal is very good at allowing us to do that. But that complexity needs to be contained.

[Typed Entity](<https://www.drupal.org/project/typed_entity>) is a module that allows you to do this. We want to keep logic close to the entity that logic affects and not scattered around in hooks. You might be altering a form related to the node or doing with access or operate on something related to an entity with a service.

In this example, Book is not precisely a node, but it contains a node of type *Book* in its $entity property. All the business logic related to *Book* node types will be contained in this class.

```php
final class Book implements LoanableInterface {
  private const FIELD_BOOK_TITLE = 'field_full_title';
  private $entity;

  public function label(): TranslatableMarkup {
    return $this->entity
      ->{static::FIELD_BOOK_TITLE}
      ->value ?? t('Title not available');
  }

  public function author(): Person {...}
  public function checkAvailability(): bool {...}

}
```

Then, in your hooks, services, and plugins, you call those methods. The result: cleaner code.

```php
// This uses the 'title' base field.
$title = $book->label();

// An object of type Author.
$author = $book->owner();

// This uses custom fields on the User entity type.
$author_name = $author->fullName();

//Some books have additional abilities and relationships
if ($book instanceof LoanableInterface) {
  $available = $book->checkAvailability() === LoanableInterface::AVAILABLE;
}
```

Business logic for books goes in the *Book* class. Business logic for your service goes in your service class. And on it goes.

If you are directly accessing field data in various places ($entity->field\_foo->value), this is a big clue you need an entity wrapper like Typed Entity.

## Focusing on entity types

Wrapping your entities does not provide organization for all of your custom code. In Drupal, however, entity types are the primary integration point for custom business logic. Intentionally organizing them will get you 80% of the way there.

Entities have a lot of responsibilities.

- They are rendered as content on the screen
- They are used for navigation purposes
- They hold SEO metadata
- They have decorative hints added to them
- Their fields are used to group content, like in Views
- They can be embedded

### Similar solutions

This concept of keeping business logic close to the entity is not unique. There is a [core patch to allow having custom classes for entity bundles](<https://www.drupal.org/project/drupal/issues/2570593>).

When you call Node::load(), the method will currently return an instance of the Node class, no matter what type the node is. The patch will allow you to get a different class based on the node type. Node::load(12) will return you an instance of the Book class, for example. This is also what the [Bundle Override](<https://www.drupal.org/project/bundle_override>) module was doing.

There are some drawbacks to this approach.

- **It increments the API surface of entity objects**. You will be able to get an instance of the Book class, but that class will still extend from the *Node* class. Your *Book* class will have all of the methods of the *Node* class, plus your custom methods. These methods could clash when Drupal is updated in the future. Unit testing remains challenging because it must carry over all the storage complexity of the *Node* class.
- **It solves the solution only partially.** What about methods that apply to many books? Or different types of books, like *SciFiBook* or *HistoryBook*. An *AudioBook*, for example, would share many methods of *Book* but be composed differently.
- **It perpetuates inheritance, even into the application space.** Framework logic bleeds into the application and business logic. This breaks the [separation of concerns](<https://en.wikipedia.org/wiki/Separation_of_concerns>). You don’t want to own the complexity of framework logic, but this inheritance forces you to deal with it. This makes your code less maintainable. We should favor composition over inheritance.

### Typed Entity’s approach

You create a plugin and associate it to an Entity Type and Bundle. These are called Typed Repositories. Repositories operate at the entity type level, so they are great for methods like findTaggedWith(). Methods that don’t belong to a specific book would go into the book repository. Bulk operations are another good example.

<drupal-entity data-embed-button="media_entity_embed" data-entity-embed-display="view_mode:media.wide" data-entity-embed-display-settings="" data-entity-type="media" data-entity-uuid="913492df-31ca-4066-90c0-6e6dccf23180" data-langcode="en"></drupal-entity>

Typed Entity is meant to help organize your project’s custom code while improving maintainability. It also seeks to optimize the developer experience while they are working on your business logic.

To maximize these goals, some tradeoffs have been made. These tradeoffs are consequences of how Drupal works and a desire to be pragmatic. While theory can help, we want to make sure things work well when the rubber meets the road. We want to make sure it is easy to use.

## Typed Entity examples

Your stakeholder comes in and gives you a new requirement: “Books located in Area 51 are considered off-limits.”

You have started using Typed Entity, and this is what your first approach looks like:

```php
/**
 * Implements hook_node_access().
 */
function physical_media_node_access(NodeInterface $node, $op, AccountInterface $account) {
  if ($node->getType() !== 'book') {
    return;
  }

  $book = \Drupal::service(RepositoryManager::class)->wrap($node);
  assert($book instanceof FindableInterface);
  $location = $book->getLocation();
  if ($location->getBuilding() === 'area51') {
    return AccessResult::forbidden('Nothing to see.');
  }

  return AccessResult::neutral();
}
```

You already have a physical\_media module, so you implement an access hook. You are using the global repository manager that comes with Typed Entity to wrap the incoming $node and then call some methods on that Wrapped Entity to determine its location.

This is a good start. But there are some improvements we can make.

We want the entity logic closer to the entity. Right now, we have logic about “book” in a hook inside physical\_media.module. We want that logic inside the *Book* class.

This way, our access hook can check on any Wrapped Entity and not care about any internal logic. It should care about physical media and not books specifically. It certainly shouldn’t care about something as specific as an “area51” string.

- Does this entity support access checks?
- If so, check it.
- If not, carry on.

Here is a more refined approach:

```php
function physical_media_node_access(NodeInterface $node, $op, AccountInterface $account) {
  try {
    $wrapped_node = typed_entity_repository_manager()->wrap($node);
  }  
  catch (RepositoryNotFoundException $exception) {
    return AccessResult::neutral();
  }

  return $wrapped_node instanceof AccessibleInterface
    ? $wrapped_node->access($op, $account, TRUE)
    : AccessResult::neutral();
}
```

If there is a repository for the $node, wrap the entity. If that $wrapped\_entity has an access() method, call it. Now, this hook works for all Wrapped Entities that implement the AccessibleInterface.

This refinement leads to better:

- Code organization
- Readability
- Code authoring/discovery (which objects implement AccessibleInterface)
- Class testability
- Static analysis
- Code reuse

## How does Typed Entity work?

So far, we’ve only shown typed\_entity\_repository\_manager()->wrap($node). This is intentional. If you are only working on the layer of an access hook, you don’t *need* to know how it works. You don’t have to care about the details. This information hiding is part of what helps create maintainable code.

But you want to write better code, and to understand the concept, you want to understand how Typed Entity is built.

So how does it work under the hood?

This is a declaration of a Typed Repository for our *Book* entities:

```php
/**
 * The repository for books.
 *
 * @TypedRepository(
 *    entity_type_id = "node",
 *    bundle = "book",
 *    wrappers = @ClassWithVariants(
 *      fallback = "Drupal\my_module\WrappedEntities\Book",
 *      variants = {
 *        "Drupal\my_module\WrappedEntities\SciFiBook",
 *      }
 *    ),
 *   description = @Translation("Repository that holds business logic")
 * )
 */
final class BookRepository extends TypedRepositoryBase {...}
```

The "wrappers" key defines which classes will wrap your Node Type. There are different types of books, so we use ClassWithVariants, which has a fallback that refers to our main *Book* class. The repository manager will now return the *Book* class or one of the variants when we pass a book node to the ::wrap() method.

More on variants. We often attach special behavior to entities with specific data, and that can be data that we cannot include statically. It might be data entered by an editor or pulled in from an API. Variants are different types of books that need some shared business logic (contained in *Book*) but also need business logic unique to them.

We might fill out the variants key like this:

```php
variants = {
  "Drupal\my_module\WrappedEntities\SciFiBook",
  "Drupal\my_module\WrappedEntities\BestsellerBook",
  "Drupal\my_module\WrappedEntities\AudioBook",
}
```

How does Typed Entity know which variant to use? Via an ::applies() method. Each variant must implement a specific interface that will force the class to implement ::applies(). This method gets a $context which contains the entity object, and you can check on any data or field to see if the class applies to that context. An ::applies() method returns TRUE or FALSE.

For example, you might have a Taxonomy field for Genre, and one of the terms is “Science Fiction.”

<drupal-entity data-embed-button="media_entity_embed" data-entity-embed-display="view_mode:media.wide" data-entity-embed-display-settings="" data-entity-type="media" data-entity-uuid="14fcdbd6-d623-413f-b9e1-47a228e69213" data-langcode="en"></drupal-entity>

### Implementing hooks

We can take this organization even further. There are many entity hooks, and Typed Entity can implement these hooks and delegate the logic to interfaces. The logic remains close to the Wrapped Entity that implements the appropriate interface.

The following example uses a hypothetical hook\_entity\_foo().

```php
/**
 * Implements hook_entity_foo().
 */
function typed_entity_entity_foo($entity, $data) {
  $wrapped = typed_entity_repository_manager()->wrap($entity);
  if (!$wrapped instanceof \Drupal\typed_entity\Fooable) {
    // if the entity not fooable, then we can't foo it
    return;
  }
  $wrapped->fooTheBar($data);
}
```

This type of implementation could be done for any entity hook.

Is this a good idea? Yes and no.

**No**, because Typed Entity doesn’t want to replace the hook system. Typed Entity wants to help you write better code that is more efficient to maintain. Reimplementing all of the hooks (thousands of them?) as interfaces doesn’t further this goal.

**Yes**, because you could do this for your own codebase where it makes sense, keeping it simple and contained. And yes, because Typed Entity does make an exception for hooks related to rendering entities.

### Rendering entities

The most common thing we do with entities is to render them. When rendering entities, we already have variants called “[view modes](<https://www.drupal.org/docs/8/api/entity-api/display-modes-view-modes-and-form-modes>)” that apply in specific contexts.

This is starting to sound familiar. It sounds like a different type of wrapped object could overlay this system and allow us to organize our code further. This would let us put everything related to rendering an entity type (preprocess logic, view alters, etc.) into its own wrapped object, called a **renderer**. We don’t have to stuff all of our rendering logic into one Wrapped Entity class.

Typed Entity currently supports three of these hooks:

- hook\_entity\_view\_alter()
- hook\_preprocess()
- hook\_entity\_display\_build\_alter()

Renderers are declared in the repositories. Taking our repository example from above, we add a "renderers" key:

```php
/**
 * The repository for books.
 *
 * @TypedRepository(
 *    entity_type_id = "node",
 *    bundle = "book",
 *    wrappers = @ClassWithVariants(
 *      fallback = "Drupal\my_module\WrappedEntities\Book",
 *      variants = {
 *        "Drupal\my_module\WrappedEntities\SciFiBook",
 *      }
 *    ),
 *    renderers = @ClassWithVariants(
 *      fallback = "Drupal\my_module\Renderers\Base",
 *      variants = {
 *        "Drupal\my_module\Renderers\Teaser",
 *      }
 *    ),
 *   description = @Translation("Repository that holds business logic")
 * )
 */
final class BookRepository extends TypedRepositoryBase {...}
```

If you understand wrappers, you understand renderers.

The *TypedEntityRendererBase* has a default ::applies() method to check the view mode being rendered and select the proper variant. See below:

<drupal-entity data-embed-button="media_entity_embed" data-entity-embed-display="view_mode:media.wide" data-entity-embed-display-settings="" data-entity-type="media" data-entity-uuid="80de445f-9a98-42cc-ae3c-0ced408188cb" data-langcode="en"></drupal-entity>

These renderers are much easier to test than individual hook implementations, as you can mock any of the dependencies.

## Summary

Typed Entity can help you make your code more testable, discoverable, maintainable, and readable. Specifically, it can help you:

- **Encapsulate** your business logic in wrappers
- Add **variants** (if needed) for specialized business logic
- Check for wrapper **interfaces** when implementing hooks/services
- Use **renderers** instead of logic in rendering-specific hooks
- Add variants per **view mode**.

All of this leads to a codebase that is easier to expand and cheaper to maintain.

<small>Photo by <a href="https://unsplash.com/@jstrippa?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">James Harrison</a> on <a href="https://unsplash.com/@jstrippa?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>
  