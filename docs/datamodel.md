### Fucked up JSON data model

At the highest level each object has some ID which instructs us how to handle the data block. An example:

    {
      // 0 indicates root definition
      "object-type": 0,
      
      // In each JSON object the content is stored in a data block
      "data": {
        // in this case it's defining known object types and their name
        // this will be referenced below as objectTypeEnum
        "0": "global-object-listing",
        "1": "person",
        "2": "company",
        "3": "evidence"
      }
    }

From this example we can also see the rough layout of each object represented in our system&mdash;`object['object-type']` & `object['data']` can be assumed to always be available.  For all `objects-type`s that are not `0` it is also safe to assume `object['data']['id']` which will be a stable id within object-type namespace.

Our top level objects are currently:

* `person`: An individual that we maintain data on
* `company`: Some organization composed of many individuals. For the time being this represents anything that is not a single person
* `evidence`: A datapoint describing one or more `person`s or `company`s; this object will contains links to the relevant entities

Currently these first level objects are where all data is rooted within our system and can be enriched by including `evidence` describing some relevant content about the parent

### Schema and Validation

TODO: Surely there is some nice JSON schema out there on the internet?  If we start taking external submissions there should be some tooling to verify that things remain in good order.  Essentially I want data model validation on pull requests.

For the time being I'll use a pseudo-schema created as I go:
#### Links
Links and related objects aren't used as a top level object but are used to connect the parent object to another.  They will be used commonly throughout the rest of the data model so isolating it seemed reasonable.

    // yes, this doesn't make any sense in JSON so let's pretend this gets
    // turned into something that the validator understands to enforce when
    // checking linkObject.link-type
    linkTypeEnum {
      // before using relationship-other we should come up with a way to
      // indicate what 'other' means; presumable by updating linkObject
      "0": "relationship-other",
      "1": "relationship-leadership",
      "2": "relationship-investor"
    }
    
    linkDestinationObject {
      "object-type": required objectTypeEnum,
      "object-id": required long
    }
  
    linkObject {
      "link-to": required linkDestinationObject,
      "link-type": required linkTypeEnum,
      // Catch-all in case we want rudimentary tags or similiar
      "link-meta": required [string]
    }

#### Person

    "data": required object {
      "id": required long,
      "name": required string,
      "bio": optional string,
      "presence": optional object {
        // required in that a consumer is assured a list will be here;
        // if no content it should be empty
        "website": required [string],
        "twitter": required [string],
      }
      "links": required [linkObject]
    }

Potentially worth noting is that I'm explicitly choosing to exclude most contact info.  The goal here is not to make people you dislke more accessible.

#### Company

    "data": required object {
      "id": required long,
      "name": required string,
      "description": optional string,
      "industry": required [string],
      "presence": optional object {
        "email": required [addressObject]
        "location": required [adressObject],
        // It may behoove us to transition to a more generalized encoding
        // for social media presence
        "website": required [string],
        "twitter": required [string],
        "facebook": required [string]
      },
      "links": required [linkObject]
    }
    
    addressObject {
      "name": required string,
      "address": required string,
      ""description": optional string
    }

#### Evidence
This is currently extremely ill-defined.  I envision this being largely collections of articles, social media action, website archives, etc.  In theory we should have some way to ensure that the data linked is stable (so not a tweet that will be deleted or some unreliable website that can be redirected) or an archived copy of whatever it is at the time of submission.  For the early stages of this project we can just accept 'evidence' submissions from known good sources.

    "data": required object {
      "id": required long,
      "meta": required[string],
      "description": optional string,
      "links": required [linkObject]
    }
