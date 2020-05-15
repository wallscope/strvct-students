
# STRVCT API Readme

The STRVCT Backend API is written in Kotlin. It serves as the bridge between the front-end (written in Vue.js with TypeScript/PUG/SCSS) and the triplestore (Blazegraph) which stores the generated vocabulary. 
All available endpoints are explained below.

In [/vocabularies](vocabularies/), there are three vocabularies, all of varying sizes, labeled as small, medium and large.

The API is available at: link TBA


# Vocabulary 

The vocabularies are all RDF vocabularies in SKOS notation. To read more, check out either [the official spec](https://www.w3.org/TR/swbp-skos-core-spec/) or [some of our blog posts](https://medium.com/wallscope) for more insight. [This one](https://medium.com/wallscope/linked-data-a-conceptual-exploration-9860a1f44d68?source=collection_home---6------7-----------------------) is particularly useful to help introduce you to Linked Data & RDF, written by our very own [@afduarte](https://github.com/afduarte)

There are many serialisations of RDF, the most human readable and the format that you are provided is `.ttl (Turtle)`. The examples in this section write out concepts in `.ttl` format. 

In short, STRVCT will generate a vocabulary where each concept (think individual) will have a set amount of properties: a name, some note, some keywords, and a "parent" concept. One concept can have one & only one parent, but many children. 

Here's the general shape of a concept in a vocabulary: 

```
<conceptURI> a skos:Concept ;
schema:keywords "comma, separated, keywords" ;
skos:prefLabel "concept name" ;
skos:note "some note about the concept" ;
skos:broader <anotherConceptUri> .
```

It's easier to think of a vocabulary as a tree-like hierarchy. Here's the general shape a vocabulary takes:

```
a
	b (child of a)
		c (child of b)
d
	f (child of d)
e
```

# Endpoints

## GET /getentities
Returns all entities from the triplestore as JSON Objects in an array. 

### Response Shape
```javascript
[{
    "name": string
    "uri": string
    "parentURI": string
    "note": string
    "keywords": string[]
}]
```

### Response example
```json
[{
    "name": "plant",
    "uri": "http://wallscope.co.uk/ontology/7bb9b4c9-e38e-46b0-ba9d-5c9094f968ab",
    "parentURI": "http://wallscope.co.uk/ontology/7bb9b4c9-e38e-46b0-ba9d-5c9094f968ac",
    "note": "photosynthesises for us",
    "keywords": [
      "leaves",
      "roots",
      "branches"
    ]
},
{
    "name": "animal",
    "uri": "http://wallscope.co.uk/ontology/7bb9b4c9-e38e-46b0-ba9d-5c9094f968a3",
    "parentURI": "http://wallscope.co.uk/ontology/7bb9b4c9-e38e-46b0-ba9d-5c9094f968a321c",
    "note": "is part of the animal kingdom",
    "keywords": [
      "pet",
      "wild",
      "cute"
    ]
},
],
```


## GET /getvocabularyfile

### Request shape:
```javascript
{ params: {
        ext: string
      }
}
```
Where ext is a file extension, either `ttl`, `rdf` or `n3`.
### Response Shape
The response from the API is a `text/plain;charset=utf-8` content type, in the form of an RDF file, the easiest to read is a .ttl file. See here: https://www.w3.org/TR/turtle/


## POST /addentity

### Request shape: 
JSON Object with the following: 
```javascript
{
    name: string;
    parentName?: string;
    parentURI?: string;
    note?: string;
    keywords?: string[];
}
  ```

The `?` denotes that the fields are optional. Note that the URI for the entitiy is generated in the backend using a random UUIDV4. If any optional fields are present, they're added to the concept, otherwise ignored.

## POST /clearstore

Posting to this endpoint simply clears out the triple store.

## POST /deleteentity

### Request shape:
JSON Object with the following: 
```javascript
{ uri: string }
```

Deletes an entity from the triple store with the URI provided. 

## POST /changedesc

### Request shape:
JSON Object with the following: 
```javascript
{
	uri: string,
	note: string
}
```

Endpoint to change the skos:note property of an entity. In this case, the uri and note are required as strings. The API then finds the entity by URI and updates the note property.


##POST /changename

### Request shape: 
JSON Object with the following:
```javascript
{
	uri: string,
	name: string,
}
```
Endpoint to change the skos:prefLabel (name) property of an entity. In this case, the uri and name are required as strings. The API then finds the entity by URI and updates the name property.


## POST /changekeywords

### Request shape:
JSON Object with the following: 
```javascript
{
	uri: string,
	keywords: string
}
```

Endpoint to change the schema:keywords property of an entity. In this case, the uri and keywords are required as strings. According to the documentation, the schema:keywords property should be a string, with each keyword separated by a comma. For example `{uri: "someURI", keywords: "key1, key2, key3"}`. 


## POST /uploadvocabulary
### Request shape:
```javascript
{ headers: { "Content-Type": "multipart/form-data" } }
//Apend a valid RDF file to form data & post it
```

The backend ingests the given RDF file, clears the store and then uploads the file's RDF to it.


