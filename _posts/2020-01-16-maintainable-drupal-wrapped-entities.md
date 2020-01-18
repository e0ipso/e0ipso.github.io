---
title: 'Maintainable Code in Drupal: Wrapped Entities'
categories:
 - drupal
tags:
 - Open source
 - Drupal
canonical_path: https://www.lullabot.com/articles/maintainable-code-drupal-wrapped-entities
---
_This is a re-post of the article I wrote for the [Lullabot blog](https://www.lullabot.com/articles/maintainable-code-drupal-wrapped-entities)._

How can more maintainable custom code in Drupal be written? Refactor it to follow SOLID software design principles. As long SOLID purity isn't pursued into an endless rabbit hole, SOLID principles can improve project maintainability. When a project has low complexity, it is worthwhile to respect these principles because they're simple to implement. When a project is complex, it is worthwhile to respect these principles to make the project more maintainable.
<!-- more -->
Wrapped entities are custom classes around existing entities that will allow custom Drupal code to be more SOLID. This is achieved in three steps:

1. Hide the entities from all custom code.
2. Expose meaningful interaction methods in the wrapped entity.
3. Implement the wrapped entities with the SOLID principles in mind.

Finally, many of the common operational challenges when implementing this pattern are solved when using the [Typed Entity](https://www.drupal.org/project/typed_entity) module.

## SOLID Principles

More information can be found about the SOLID principles in [the Wikipedia article](https://en.wikipedia.org/wiki/SOLID), but here’s a quick summary:


> **S**ingle responsibility principle: A [class](https://en.wikipedia.org/wiki/Class_(computer_programming)) should only have a single responsibility, that is, only changes to one part of the software's specification should be able to affect the specification of the class.
>
> **O**pen–closed principle: Software entities should be open for extension, but closed for modification.
>
> **L**iskov substitution principle: Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program. This is often referred to as design by contract.
>
> **I**nterface segregation principle: Many client-specific interfaces are better than one general-purpose interface.
>
> **D**ependency inversion principle: One should "depend upon abstractions, [not] concretions."

Here, the focus will be on the wrapped entity pattern. In this pattern, it's discovered that PHP classes can be created that wrap Drupal entities to encode a project's business logic and make the code more SOLID. This is the logic that makes a project unique in the world and requires custom code to be written in the first place because it’s not completely abstracted by libraries and contributed modules.

First, understand the theory, and then look at the actual code.

## Building a Library Catalog

To help exemplify this pattern, imagine being tasked with building the catalog for a local library. Some of these examples may be solved using a particular contrib module. However, this exercise focuses on custom code. 

### Entities Are Now Private Objects

Business logic in Drupal revolves around entities and the relationships between them. Often, the business logic is scattered in myriad hooks across different modules. The maintainability of custom code can be improved by containing all this custom logic in a single place. Then, the myriad of hooks can make simple calls to this central place.

The business logic is often contained around entities. However, entities are very complex objects in Drupal 8. They’re already associated with many features; they can be moderated, rendered, checked for custom access rules, part of lists, extended via modules, and more. How can the complexity be managed with the business logic when already starting with such a complex object? The answer is by hiding the entity from the custom code behind a [facade](https://en.wikipedia.org/wiki/Facade_pattern). These objects are called wrapped entities.

In this scenario, only the facade can interact directly with the entity at any given time. The entity itself becomes a hidden object that only the facade communicates with. If `$entity->field_foo` appears anywhere in the code, that means that the wrapped entity needs a new method (or object) that describes what's being done.

### Wrapped Entities

In the future, [it may be possible](https://www.drupal.org/project/drupal/issues/2570593) to specify a custom class for entities of a given bundle. In that scenario, `Node::load(12)` will yield an object of type `\Drupal\physical_media\Entity\Book`, or a `\Drupal\my_module\Entity\Building`. Nowadays, it always returns a `\Drupal\node\Entity\Node`. While this is a step forward, it’s better to hide the implementation details of entities. Adding the business logic to entity classes increases the API surface and is bad for maintenance. This approach also creates a chance of naming collisions between custom methods and other added-to entities in the future.

A wrapped entity is a class that can access the underlying entity (also known as "the model") and exposes some public methods based on it. The wrapper is not returned from a `Node::load` call, but created on demand. Something like `\Drupal::service(RepositoryManager::class)->wrap($entity)` will return the appropriate object `Book`, `Movie`, etc. In general, a `WrappedEntityInterface`.

A wrapped entity combines some well known [OOP design patterns](https://www.lullabot.com/articles/using-the-template-method-pattern-in-drupal-8):

- [Factory pattern](https://www.lullabot.com/articles/introduction-to-the-factory-pattern).
- [Facade pattern](https://en.wikipedia.org/wiki/Facade_pattern).

In this catalog example, the class could be (excuse the code abridged for readability):

```language-php
    interface BookInterface extends
      LoanableInterface,
      WrappedEntityInterface,
      PhysicalMediaInterface,
      FindableInterface {}
    
    class Book extends WrappedEntityBase implements BookInterface {
    
      private const FIELD_NAME_ISBN = 'field_isbn';
    
      // From LoanableInterface.
      public function loan(Account $account, DateTimeInterval $interval): LoanableInterface { /* … */ }
      // From PhysicalMediaInterface.
      public function isbn(): string {
        return $this->entity->get(static::FIELD_NAME_ISBN)->value;
      }
      // From FindableInterface.
      public static function getLocation(): Location { /* … */ }
    
    }
```

More complete and detailed code will follow. For now, this snippet shows that using this pattern can turn a general-purpose object (the entity) into a more focused one. This is the S in **S**OLID. There are commonalities that group methods together when considering the semantics of each method written. Those groups become interfaces, and all of a sudden, it's apparent interface segregation has been implemented. **S**OL**I**D. With those granular interfaces, things like the following can be done:

```language-php
array_map(
  function (LoanableInterface $loanable) { /* … */ },
  $loanable_repository->findOverdue(new \DateTime('now'))
);
```

Instead of:

```language-php
array_map(
  function ($loanable) {
    assert($loanable instanceof Book || $loanable instanceof Movie);
    /* … */
  },
  $loanable_repository->findOverdue(new \DateTime('now'))
);
```

Coding to the object's capabilities and not to the object class is an example of dependency inversion. **S**OL**ID**.

### Entity Repositories

Wrapped entity repositories are services that retrieve and persist wrapped entities. In this case, they're also used as factories to create the wrapped entity objects, even if that is not very SOLID. This design decision was made to avoid yet another service when working with wrapped entities. The aim is to improve DX.

While a wrapped entity `Movie` deals with the logic of a particular node, the `MovieRepository` deals with logic around movies that applies to more than one movie instance (like `MovieRepository::findByCategory`), and the mapping of `Movie` to database storage.

### Further Refinements

Sometimes, it's preferred to not have only one type per bundle. It would be reasonable that books tagged with the *Fantasy* category upcast to `\Drupal\physical_media\Entity\FantasyBook`. This technique allows for staying open for extension (add that new fantasy refinement) while staying closed for change (`Book` stays the same). This takes O into account. **SO**L**ID**. Since `FantasyBook` is just a subtype of `Book`, this new extension can be used anywhere where a book can be used. This is called the Liskov substitution principle, our last letter. **SOLID**.

Of course, the same applies to heterogeneous repositories like `LoanableRepository::findOverdue`.

## Working On The Code

This section shows several code samples on how to implement the library catalog. To better illustrate these principles, the feature requirements have been simplified. While reading, try to imagine complex requirements and how they fit into this pattern.

### Typed Entity Module

_I ported the [Typed Entity](https://www.drupal.org/project/typed_entity) module to Drupal 8 to better support this article, and it has helped a lot with my Drupal 7 projects. I decided to do a full rewrite of the code because I have refined my understanding of the problem during the past four years._

The code samples leverage the base classes in [Typed Entity](https://www.drupal.org/project/typed_entity) and its infrastructure. [The full code samples can be accessed in this repository](https://github.com/e0ipso/physical_media). This repo contains the latest code and corrections.

### Hands-On

The first focus will be to create a wrapped entity for a book. This facade will live under `\Drupal\physical_media\WrappedEntities\Book`. The class looks like this (for now):


```language-php
namespace Drupal\physical_media\WrappedEntities;
    
use Drupal\physical_media\Location;
use Drupal\typed_entity\WrappedEntities\WrappedEntityBase;
    
class Book extends WrappedEntityBase implements LoanableInterface, PhysicalMediaInterface, FindableInterface {
    
  const FIELD_NAME_ISBN = 'field_isbn';
  const FIELD_NAME_LOCATION = 'field_physical_location';
    
  // From PhysicalMediaInterface.
  public function isbn(): string {
    return $this->getEntity()->get(static::FIELD_NAME_ISBN)->value;
  }
    
  // From FindableInterface.
  public function getLocation(): Location {
    $location = $this->getEntity()->get(static::FIELD_NAME_LOCATION)->value;
    return new Location(
      $location['building'],
      $location['floor'],
      $location['aile'],
      $location['section']
    );
  }
   
}
```    

Then, the repository will be registered in the service container. The `physical_media.services.yml` contains:

```language-yaml
services:
  physical_media.typed_entity.repository.book:
    # If you don't have custom logic for your repository you can use the base
    # class and save yourself from writing another empty class.
    # class: Drupal\typed_entity\TypedRepositories\TypedEntityRepositoryBase
    class: Drupal\physical_media\TypedEntityRepositories\BookRepository
    parent: Drupal\typed_entity\TypedRepositories\TypedEntityRepositoryBase
    public: true
    tags:
      -
        name: typed_entity_repository
        entity_type_id: node
        bundle: book
        wrapper_class: Drupal\physical_media\WrappedEntities\Book
```

Important bits are:

- Specify the `parent` key to inherit additional service configuration from the contrib module.
- If there's no reason to have a custom repository, the base class under the `class` key can be used.
- Add the service tag with all the required properties: 
    - `name`: this should always be `typed_entity_repository`.
    - `entity_type_id`: the entity type ID of the entities that will be wrapped. In this case, books are nodes.
    - `bundle`: the bundle. The bundle can be omitted only if the entity type has no bundles, [like the user entity](https://git.drupalcode.org/project/typed_entity/blob/8.x-1.x/modules/typed_entity_example/typed_entity_example.services.yml#L15-23).
    - `wrapper_class`: the class that contains the business logic for the entity. This is the default class to use if no other variant is specified. Variants will be covered later.

Once these are done, the wrapped entity can begin integration into the hook system (or wherever it pertains). An integration example that restricts access to books based on their physical location could be:


```language-php
<?php
use Drupal\Core\Access\AccessResult;
use Drupal\Core\Session\AccountInterface;
use Drupal\node\NodeInterface;
use Drupal\physical_media\WrappedEntities\FindableInterface;
use Drupal\typed_entity\RepositoryManager;
    
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

Better yet, the `Book` could be refactored and that leaking business logic (the one checking for specific buildings) could be put into it, and then a reasonable interface could be implemented for it (favor existing ones) like `Book … implements \Drupal\Core\Access\AccessibleInterface`.

```language-php
…
use Drupal\Core\Access\AccessibleInterface;
use Drupal\Core\Access\AccessResult;

class Book … implements …, AccessibleInterface {
  …
  // From AccessibleInterface.
  public function access($operation, AccountInterface $account = NULL, $return_as_object = FALSE) {
    $location = $this->getLocation();
    if ($location->getBuilding() === 'area51') {
      return AccessResult::forbidden('Nothing to see.');
    }
    return AccessResult::neutral();
  }
    
}
```

While evolving the hook into:

```language-php
function physical_media_node_access($node, $op, $account) {
  return $node->getType() === 'book' ?
    \Drupal::service(RepositoryManager::class)
      ->wrap($node)
      ->access($op, $account, TRUE)
    : AccessResult::neutral();
}
```
 
The code can still be improved to remove the check on `'book'`. Books are checked for access because of knowledge about the business logic. That leak can be avoided by trying to code to object capabilities instead ([D](https://en.wikipedia.org/wiki/Dependency_inversion_principle) in SOLID).

```language-php
function physical_media_node_access($node, $op, $account) {
  try {
    $wrapped_node = \Drupal::service(RepositoryManager::class)->wrap($node);
  }
  catch (RepositoryNotFoundException $exception) {
    return AccessResult::neutral();
  }
  return $wrapped_node instanceof AccessibleInterface
    ? $wrapped_node->access($op, $account, TRUE)
    : AccessResult::neutral();
}
```

After that, the hook can remain the same when access control is implemented in movies. Not having to trace the whole codebase for potential changes when new features are added or existing ones are changed is a big win for maintainability.

### Handling Complexity

As code grows in complexity, making it maintainable becomes more difficult. This is a natural consequence of complexity so it's best to be pragmatic and not take the SOLID principles as a hard rule. They exist to serve a purpose, not the other way around.

There are some ways to contain complexity. One is to avoid passing other entities into the wrapper methods by using wrapped entities instead. This creates more discipline in making the business logic explicit. [This example](https://git.drupalcode.org/project/typed_entity/blob/8.x-1.x/src/WrappedEntities/WrappedEntityBase.php#L50-67) gets the wrapped user entity as the author of an article.

```language-php
public function owner(): ?WrappedEntityInterface {
  $owner_key = $this->getEntity()->getEntityType()->getKey('owner');
  if (!$owner_key) {
    return NULL;
  }
  $owner = $this->getEntity()->{$owner_key}->entity;
  if (!$owner instanceof EntityInterface) {
    return NULL;
  }
  $manager = \Drupal::service(RepositoryManager::class);
  assert($manager instanceof RepositoryManager);
  return $manager->wrap($owner);
}
```

Another way to contain complexity is by splitting wrapped entities into sub-classes. If having several methods that don’t apply to some books, variants can be beneficial. Variants are wrappers for a given bundle that are specific to a subgroup. 

The Typed Entity Examples submodule contains an example on how to create a variant. In that example, the repository for articles [hosts variant conditions](https://git.drupalcode.org/project/typed_entity/blob/8.x-1.x/modules/typed_entity_example/src/TypedRepositories/ArticleRepository.php#L26-28). Calls to `RepositoryManager::wrap($node)` with article nodes now yield  [`Article`](https://git.drupalcode.org/project/typed_entity/blob/8.x-1.x/modules/typed_entity_example/src/WrappedEntities/Article.php) or [`BakingArticle`](https://git.drupalcode.org/project/typed_entity/blob/8.x-1.x/modules/typed_entity_example/src/WrappedEntities/BakingArticle.php) depending on whether or not the node is tagged with the term `'baking'`. The contrib module [comes with a configurable condition](https://git.drupalcode.org/project/typed_entity/blob/8.x-1.x/src/WrappedEntityVariants/FieldValueVariantCondition.php) to create a variant based on the content of a field. That is the most common use case, but any condition can be written. If having a different wrapper is preferred for Wednesday articles, that (odd) condition can be written implementing [`VariantConditionInterface`](https://git.drupalcode.org/project/typed_entity/blob/8.x-1.x/src/WrappedEntityVariants/VariantConditionInterface.php).

## Summary

Decades-old principles were highlighted as still being relevant to today’s Drupal projects. SOLID principles can guide the way custom code is written and result in much more maintainable software.

Given that entities are one of the central points for custom business logic, the wrapped entities pattern was covered. This facade enables making the business logic explicit in a single place while hiding the implementation details of the underlying entity.

Finally, the Typed Entity module as a means of standardization across projects when implementing this pattern was explored. This tool can only do so much because ultimately, the project’s idiosyncrasies cannot be generalized; each project is different. However, it is a good tool to help promote more maintainable patterns.
