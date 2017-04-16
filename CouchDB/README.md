# CouchDB

## Why CouchDB?

* Open source
* Built in REST API
* Easy setup
* Can handle validation and authentication

## Setup

1. Run `docker-compose up` to start an instance of CouchDB 2.0 running locally on port 5984
1. Open your browser to `http://localhost:5984/_utils` to access the Fauxton admin panel.
1. Navigate to `Setup` > `Configure Single Node` and specify a username and password. Leave the IP and port.

## A quick tour of CouchDB

The following can all be completed via curl requests against the REST API.

1. Create A database
  * 'Databases'>'Create Database'> Let's name our's "catalog"
1. Insert some documents
  * 'All Documents'>'+'>'New Doc'
  * Here is some starting data. Typically, you'd let CouchDB create the `_id`, but this makes it easier for us to quickly create some relationships

  Components:

  ```
  {
    "_id": "735174c9bbfec0115fa1e65e630035ed",
    "type": "component",
    "name": "d20 Black",
    "sides": "20",
    "appearance": "black"
   }
  ```

  ```
  {
    "_id": "735174c9bbfec0115fa1e65e63011d30",
    "_rev": "1-6a4c2db6ebe043ab37bd4ff06b55de97",
    "type": "component",
    "name": "d4 Red",
    "sides": "4",
    "appearance": "red"
   }
  ```
  Games:

  ```
  {
    "_id": "735174c9bbfec0115fa1e65e6301486e",
    "type": "game",
    "name": "Dungeons & Dragons Starter Set",
    "vendor": "Wizards of the Coast",
    "contents": [
      {
        "qty": 2,
        "id": "735174c9bbfec0115fa1e65e630035ed"
      },
      {
        "qty": 4,
        "id": "735174c9bbfec0115fa1e65e63011d30"
      }
    ]
  }
  ```
  Game Systems:

  ```
  {
    "_id": "735174c9bbfec0115fa1e65e63016382",
    "name": "Dungeons & Dragons",
    "type": "game-system",
    "vendor": "Wizards of the Coast",
    "contents": [
      {
        "qty": 3,
        "id": "735174c9bbfec0115fa1e65e630035ed"
      },
      {
        "qty": 5,
        "id": "735174c9bbfec0115fa1e65e63011d30"
      },
      {
        "qty": 1,
        "id": "735174c9bbfec0115fa1e65e6301486e"
      }
    ]
  }
  ```

  ```
  {
    "_id": "735174c9bbfec0115fa1e65e63019d6e",
    "name": "Pathfinder",
    "type": "game-system",
    "vendor": "Pazio",
    "contents": [
      {
        "qty": 10,
        "id": "735174c9bbfec0115fa1e65e630035ed"
      },
      {
        "qty": 4,
        "id": "735174c9bbfec0115fa1e65e63011d30"
      }
    ]
  }
  ```
  So now there are 2 game-systems, which have contents that are comprised of components and games.

1. Query our documents

  All documents are automatically displayed at `http://localhost:5984/_utils/#/database/catalog/_all_docs` but you can also use curl to retrieve `curl http://localhost:5984/catalog/_all_docs`

  By default, CouchDB only returns the IDs and metadata, if you want to include the documents, append `?include_docs=true` to your curl request. Or, in the Fauxton admin panel, select the "Documents" checkbox up top.

1. Create a view (all game-systems)

  * CouchDB uses map functions to build views. Create one by selecting "Design Documents">"+">"New View"
  * Create a new design document to aggregate our views. In the input box labeled `_design/`, name your document "contents".
  * Name the index 'game-systems'
  * Create the following map function to view only game Systems

  ```
  function (doc) {
    if(doc.type === 'game-system'){
      emit(doc._id, null);
    }
  }
  ```
1. Query our view

  * `curl http://localhost:5984/catalog/_design/contents/_view/game-systems?include_docs=true`

  * This view has created an index on the key (the game-system id), which it will maintain for performance.

1. Build a more complex view (key: game, value: contents)

  * Create a new view, this time indexing all of our components and their 'parent'. The index will be an array of the parent ID, and an index, and the value will be the component object (instead of an ID)
  * Name the view 'owners' and use the following map function:

  ```
  function (doc) {
    if(doc.name && doc.contents) {
      for (var i in doc.contents) {
        var content = doc.contents[i];
        emit([doc._id, i], {"_id": content.id, "content": content});
      }
    }
  }
  ```

  * This is our "Join" statement. It returns a row for each component in each object that has 'contents'.
  * This map function first checks if there are contents. It then loops through each element and extracts the content. It emits key value pairs as follows: The key is an array of the owner's ID, and the index (to ensure uniqueness), the value is an object containing an `_id` (which will become apparent soon), and the entire content object.
  * If you query without `include_docs=true`, you'll only see the id of the content, as `value.content.id`. If we do include the docs, then we're additionally returned a doc that matches our content. CouchDB uses the `_id` field in the value to determine which document to attach.

1. Search on our view

  * The following query fetches all the contents for the D&D game-system: `curl -X GET localhost:5984/catalog/_design/contents/_view/all-contents-and-amounts?include_docs=true -G --data-urlencode start_key='["735174c9bbfec0115fa1e65e63001edc",""]' --data-urlencode end_key='["735174c9bbfec0115fa1e65e63001edc",{}]' | python -mjson.tool`

## What's missing

## Summary
