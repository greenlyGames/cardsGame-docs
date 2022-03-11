---
id: entities-and-traits
title: Entities & Traits
sidebar_position: 3
---

# Entities &amp; Traits

Entities are game elements or "virtual" containers. Entities are made up of Traits.

> TODO: diagram showing all traits and example how are classicCard and Pile made up

`@cardsgame/server` contains many basic entities and containers, but you can create your own.

:::tip Remember

This page explains entities on server-side. It's up to you to define how these entities look like on the client-side.

:::

## Entities

### ClassicCard

### Container

### Deck

### Grid

### Hand

### Line

### Pile

### Spread

## Traits

### BoxModelTrait

Currently unused

### ChildTrait

**Important trait**

Entity can become a child of any other container/parent.

### FlexyTrait

Defines entity as flexible in size. Contains props useful to create Flex Box in web browser:

- `alignItems`
- `justifyContent`
- `directionReverse`

### IdentityTrait

:::caution Required trait

Every entity has to have it. Game state keeps track of every entity in game by their ID's

:::

> TODO: I might just bake that trait into base Entity class...

### LabelTrait

Adds `name` and `type` properties, useful for querying on server-side and for choosing client-side component.

> TODO: I might move `type` to base Entity class, seems I'm using `type` everywhere anyway

### LocationTrait

Adds `x`, `y` and `angle` to entity, describing its location and rotation, relative to its parent.

### OwnershipTrait

Entity with this trait can have a reference to `Player`, who "owns" this entity.

### ParentTrait

**Important trait**

Entity will behave as container to hold all other entities with `ChildTrait`.

Has many methods for adding, fetching, removing and manipulating order of children.

Can behave as:

- an array - all children are indexed next to each other
- a map - there can be empty spaces between children

### SelectableChildrenTrait

Used on **container** - its children can now be selected by players.

Holds indexes of selected children and in which order were these chosen.

### TwoSidedTrait

Adds `faceUp` property and `flip*()` methods. The entity now has two sides. Like a card or coin.

## Create custom entity

```
applyTraitsMixins()
```
