# Building a game catalog with various NoSQL databases

We want to build a NoSQL database that will allow us to store various types of games, that could have many components, which may be other game components.

## Data Set

We can have games, which can have multiple forms of content (one to many). Game content may include bare components (i.e. Dice), or components that are actually other games (i.e. A starter set, which has it's own contents). Games can be of different categories and may have distinctly different attributes that we would like to avoid storing in a traditional relational way.

For example, we can have the following game:

```
{
  "name": "Dungeons & Dragons",
  "type": "game-system",
  "vendor": "Wizards of the Coast",
  "contents": [
    {"type":"component", "name":"d20", "color":"black", "sides":20},
    "type":"component", "name":"Dungeon Master's Guide", "vendor":"Wizards of the Coast"},
    {"type":"game", "name":"Dungeons & Dragons Starter Set", "vendor":"Wizards of the Coast", "contents":[
      {"type":"component", "name":"d20", "color":"black", "sides":20}
      {"type":"component", "name":"d4", "color":"red", "sides":4}
    ]}
    {
  ]
}
```

You'll notice that the top level object (Dungeons & Dragons) is considered a `game-system`, which we can think of as a collection of various game pieces we may use to play. You'll see contents like dice and the Dungeon Master's guide, which are single items. We also have a `game`, which can belong to the game system, but also have it's own components (which may be shared by other entities).

While this is a contrived example, we're trying to demonstrate a somewhat complex relationship, where the top-level entity may contain both 'base' items (i.e. `component`s), as well as other 'collections' (i.e `game`s). In reality, we could would like to support many different entity types, which may or may not contain other entities.

## The SQL Relational Way

We certainly can represent this in a relational SQL database. We would have a table of `game-systems`, `games`, and `components`. In order to express these relationships, we need to build relational tables in order for us to represent all the various contents a game can have. We can make something like a `game-systems-components` table. Our `game-systems` table will have a one to many relationship to this table, which in turn, will have a many-to-one relationship with components. We will need to do this for each relationship we need to define between an entity that has contents, and the content types it supports. In this example, this only results in 6 tables, which is rather manageable, but consider the fact that as we add more entities, we have to now build out additional relationship tables, and manage those relationships too.

## Is NoSQL the answer?

It depends on where you want your pain points. This hopefully serves as an example of the pros and cons of each approach. Most non-graph NoSQL databases aren't designed to handle relational data. In fact, that's one of the distinguishing features of most NoSQL databases. But, they do give us the benefit of not having to define and maintain these relationships up front, allowing us to define these relationships as we need them.

## Viewing this tutorial

Each subdirectory should contain everything needed to run these examples on your machine. This does assume you have Docker running locally, as well as have access to bash.

## Additional Links

[Martin Fowler - Intro to NoSQL Talk](https://www.youtube.com/watch?v=qI_g07C_Q5I)
