---
title: Do define custom data types in Drupal
categories:
  - web-development
  - notes
  - drupal
tags:
  - Drupal Development
---
Today I was included in a conversation in Drupal Slack that reminded me how
important it is to take the time to learn one tool well.

In this case the issue was solved by creating a custom data type. This is the conversation log.
<!-- more -->
> **wimleers (he/him)  6 hours ago**
> So you want to normalize the data for the `administrative_area` property in the address field type differently.
> You totally can do this!
> JSON:API doesn’t allow you to customize the `@FieldType`-level normalizer (because it would easily result in not complying with the JSON:API spec). But it does allow you to customize the @DataType-level normalizer!
> In this case, that’s this bit in https://git.drupalcode.org/project/address/blob/8.x-1.x/src/Plugin/Field/FieldType/AddressItem.php:

```
$properties['administrative_area'] = DataDefinition::create('string')
       ->setLabel(t('The top-level administrative subdivision of the country.'));
```

> That’s currently using the string data type.
> But that’s kind of a problem: this is not just a string, it’s a meaningful string. So we’ll need to subclass `\Drupal\Core\TypedData\Plugin\DataType\StringData` : we’ll want something like AdministrativeAreaData extends StringData.
> Then we can change the property definition to `DataDefinition::create('administrative_area')` . And once we have that, we can create a normalizer just for that — and best of all: it will work for both `rest` and `jsonapi` :slightly_smiling_face:
>
> **wimleers (he/him)  6 hours ago**
> @mglaman @bojanz ^^
>
> **wimleers (he/him)  6 hours ago**
> @gabesullice @e0ipso ^^
>
> **bojanz  6 hours ago**
> Sounds like a good topic for the Centarro blog :slightly_smiling_face:
>
> **e0ipso:beach_with_umbrella:  6 hours ago**
> Alternatively you can write a Field Enhancer when you use JSON:API Extras
>
> **e0ipso:beach_with_umbrella:  6 hours ago**
> Even if Wim's recommendation is more convoluted, I do think it's the appropriate solution. A Field Enhancer would work for this, but it's meant for something else.
>
> **wimleers (he/him)  6 hours ago**
> @bojanz I believe that Drupal core just also sets a very bad precedent. It doesn’t distinguish between different data types at all
>
> **wimleers (he/him)  6 hours ago**
> @bojanz It feels much more logical/natural to add validation constraints to meaningful strings, i.e. strings with a particular meaning, semantic. Adding validation constraints to arbitrary string data is weird, isn’t it?
>
> **e0ipso:beach_with_umbrella:  6 hours ago**
> @wimleers (he/him) there is UUID and Date. Unless I'm miss-remembering.
>
> **wimleers (he/him)  6 hours ago**
> Yep, uuid is pretty much the only one that does this correctly
>
> **e0ipso:beach_with_umbrella:  6 hours ago**
> But it's a wildly underused feature for sure. I have never done this for client code.
>
> **wimleers (he/him)  6 hours ago**
> And the same is true in config land btw.

