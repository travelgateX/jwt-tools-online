# jwt-tools-online

It's an APIREST tool with functions that complete the use of the go-jwt-tools library (and its equivalents in other languages).

## Introduction to IAM

The IAM is the authentication project of TravelgateX, the IAM manages the permissions that a user has configured in the system. The TravelgateX teams that have their projects integrated with the IAM must comply with the following considerations:

* Each resource that a user can access must be related to a group (or product) of the IAM.
* There can be no resources accessed by the user orphans (not related to an IAM group/product).
* The special permission "all" means that the user has permissions on ALL ORGANIZATIONS and their subgroups. TEAM's and ROOT groups are out of this consideration.

In order to interpret these permissions in a simple way, support tools (libraries) have been implemented to correctly interpret a user's Bearer (generated token by the IAM)

* go-jwt-tools; [https://github.com/travelgateX/go-jwt-tools](https://github.com/travelgateX/go-jwt-tools)
* jwt-tool-dotnet: [https://github.com/travelgateX/jwt-tool-dotnet](https://github.com/travelgateX/jwt-tool-dotnet)
* node-jwt-tools: [https://github.com/travelgateX/node-jwt-tools](https://github.com/travelgateX/node-jwt-tools)

These libraries have a main function called validGroups (or hasPermission depending on the implementation) that returns a list of groups to which the user has permissions answering the question: in which groups, user X (bearer), has permissions to api Y (e.g. iam), resource Z (e.g. grp) and permission P (e.g. viewer)?

## "How do I integrate IAM into my TravelgateX product?"

When a user creates a new resource of your product he has to relate to an existing group in the IAM and keep the relationship (and group type) persistently in your system for future reference. In order to know the type of IAM group that is being saved you should consult the IAM, **this is where jwt-tools-online can help you**.

The hasPermissions function returns you if the bearer has permissions to query that resource as well as additional information from the IAM group so you can save it to your system. We strongly recommend its use.

## Instructions for use

The API has two main functions:

* **validGroups**: equivalent to the validGroups of _go-jwt-tools_, does not query the IAM or any other TravelgateX resource, only interprets the Bearer provided.
  * Request: /api/v1/auth/validgroups?api=iam&rsc=grp&per=r
  * Answer:

    ```json
    {
      "Status": 200,
      "validGroups": [
          "orgx",
          "HotelX_0000"
      ]
    }
    ```

* **hasPermission**: is equivalent to validGroups, but also consults the IAM group provided in the query and returns group information.
  * Request: /api/v1/auth/haspermission?api=iam&rsc=grp&per=r&grp=testx
  * Answer:

    ```json
    {
        "Status": 200,
        "hasPermission": true,
        "validGroups": [
            "orgx",
            "HotelX_0000."
            "testx"
        ],
        "group": {
            "id": "12286",
            "code": "testx",
            "type": "ORG"
        }
    }
    ```

## Parameters

* validGroups:
  * **api**: code of the api in the IAM, examples: ["iam", "entities", "hubgra", "stats" ...]
  * **rsc**: resource code in the IAM, examples: ["grp", "acc", "hot", "bok" ...]
  * **per**: permission code in the IAM, valid options: [c, r, u, d, x, a, f]
* hasPermission:
  * api, rsc, per
  * **grp**: group code in the IAM

## Unit tests

```bash
go test ./...
```

## Final Considerations

It's recommended not to give an intensive use to the hasPermission instruction since it consults the IAM from behind and can get to saturate it, so we recommend that each project keeps the relationship of its resource with the IAM group and type.

It's recommended to use the hasPermission instruction in the mutations of your system to obtain the relevant information of the IAM group and from there, in the consultations, to work with that relation.

## TODO

* Add cache layer to IAM group transaction
* Add cache layer to go-jwt-tools parser
* Remove configuration files from the git repository