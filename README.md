# SpringBoot-API
Source: https://rapidapi.com/blog/how-to-build-an-api-with-java/ 

## Learning Goals
1. Review Java
2. Restful API Design
3. Documenting REST API using Swagger

## Introduction
- **Representational State Transfer (REST)**: set of architectural constraints (not a protocol or standard) --> When client request is made via a RESTful API, it transfers a representation of the state of the resource to the endpoint via HTTP (several formats include JSON, HTML, Python, PHP, or plain text). 
- **API**: set of definitions and protocls for building and integrating application software (contract between information provider and a user)
- RESTful API criteria: 
    - client-server architecture made up of clients, servers, and resources, where requests are managed through HTTP
    - **Stateless** client-server communication: no client info is stared between get requests and each request is separate and unconnected
    - Cacheable data that streamlines client-server interactions
    - Unifrorm interface between components so that info is transferred in a standard form
        - resource requests are indentifiable and separate from representations sent to the client
        - resources can be manipulated by the client via the representation they receive 
        - self-descriptive messages returned to client have neough info to describe how client should process it
        - hypertext/hypermedia is availabled (after accessing resource, the client should be able to use hyperlinks to find all other currently available actions they can take)
    - Layered system that organizes different servers (security, load-balancing, etc) involved in the retrieval of requested info invisible to the client

## REST API Design
- Server provides a service (for example, it could process and store an image and its associated metadata along with info about the client posting the image)
- We can make this service availabe to external systems by allowing them to upload and manipulate images and maybe data associated with the images.
- We can create a simple, widely-accepted, robust API that clients can use to manipulate the resources in our system 
- The backend system is usually a black box. So when client makes an API request, the implementation details of how the server handles it is abstracted away

### API Design Tasks
- Determine the resources
- Create a resource model
- Formalize the resource model as an object model
- Create JSON schemas of the resources
- Write a list of actions to be performed on the resources
- Translate the object model into URLs
- Map the actions to HTTP methods and query parameters (query parameters are attached to the end of a URL that help define filter results or specify action. Like if I want to find an image posted by a particular user, the query parameter might by the user's ID)

#### Determine Resources
- **WallpaperArtist**: a person described by a collection of metadata who has one or more wallpaper images
- **WallpaperImage**: an image described by a collection of metadata that belongs to one and only one WallpaperArtist
- **WallpaperMetadata**: a key/value pair that expresses some aspect of a WallpaperImage or WallpaperArtist

#### Resource Model
- **Collection**: one or more resources of the same type. A collection can have sub-collections, and a resource can have sub-resources
- **WallpaperArtist collection** has one more more WallpaperArtist resources
- **WallpaperArtist resources** consist of a WallpaperImage collection and a WallpaperMetadata collection
- **WallpaperImage collection** is one or more WallpaperImage resources, each with a corresponding WallpaperMetadata
- **WallpaperMetadata collection** is one or more WallpaperMetadata resources

#### Object Model
- **WallpaperArtist**:
    - artistId:String
- **WallpaperImage**:
    - imageId:String
    - format: String
    - path: String
    - reference to WallpaperArtist
- **WallpaperMetadata**:
    - name: String
    - value: String
    - reference to WallpaperImage
    - reference to WallpaperArtist (or this reference can be referenced through the image?)

#### Design Rules
- RPC vs RESTful 
    // using a verb - RPC Style
    http://www.nowhere.com/imageclient/getClientsById
    // using a noun - RESTful
    http://www.nowhere.com/imageClient/{client-id}
- HTTP Request Methods
    - **GET**: Retrieve a resource 
    - **POST**: Create a new resource
    - **PUT**: Fully update an existing resource
    - **PATCH**: Modify an existing resource
    - **DELETE**: Delete 
    - **HEAD**: Receive resource metadata
    - **OPTIONS**: Receive supported HTTP request methods

#### JSON Data Representation
- JSON schema allows JSON document validation. It provides an explicit description of your API's JSON data. 
- Create new file named WallpaperImage.schema.json
    {
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "WallpaperImage",
    "description": "A image from Image catalog",
    "type": "object",
    "properties": {
        "image-id": {
        "description": "The unique identifier for an image.",
        "type": "string"
        },
        "image-format": {
        "description": "Format of image (gif, png, jpeg, etc.).",
        "type": "string"
        },
        "image-path": {
        "description": "Path/URL to the image data.",
        "type": "string"
        },
        "meta-data": {
        "description": "Metadata item describing resource.",
        "type": "array",
        "items": {
            "type": "object",
            "title": "WallpaperMetadata",
            "description": "The meta data object comprising the array.",
            "properties": {
            "name": {
                "type": "string",
                "description": "Meta data element property name."
            },
            "value": {
                "type": "string",
                "description": "Meta data element property value."
            }
            }
        }
        }
    }
    }

- Create new file named WallpaperClient.schema.json
    {
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "WallpaperClient",
    "description": "A Client from Image catalog",
    "type": "object"
    }


#### Modeling the Resurce URIs
- `GET /clients/{client-id}`
- `GET /images/{image-id}`
- `GET /clients?(meta-data=<name>=<value>&)*`
    - `<x>` notation, signifies a literal value
    - ()* repeat statements in parenthesis n times
- `GET /images?(meta-data=<name>=<value>&)*`
- `GET /clients/{client-id}/images?(meta-data<name>=<value>&)*`
- `POST /clients     *body includes <client-json-data)>`
- `POST /clients/{client-id}/images   *body includes <image-json-data> && <binary-data>`
- `PUT /clients/{client-id}      *body includes <client-json-data>`
- `PUT /images/{image-id}      *body includes <image-json-data> && <binary-data>`
- `PATCH 	/clients/{client-id}      *body includes <client-json-data>`
- `PATCH 	/images/{image-id}     *body includes <image-json-data>`
- `DELETE /clients/{client-id}`
- `DELETE /images/{image-id}`

#### Coding the API
- We'll be using Java with Spring Boot and the Jersey framework


