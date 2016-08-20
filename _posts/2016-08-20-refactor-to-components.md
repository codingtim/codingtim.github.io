---
title: "Refactoring to components."
tags:
  - java
  - design
---

This code bugged me for some time now and I finally had some time to come up with a better solution. The code comes from one of the projects I work on but I removed the domain specific names and omitted some details. In the example only the service layer will be tackled. The REST controller, database and gateway implementation are not relevant to this refactor.

This piece of code is responsible for updating an Entity. An Entity consists of a title, description and an image. During the update the code should do the following:

- Retrieve the Entity from the Repository.
- Update the Entity with the new title, description and image.
- Upload the new image to an ImageStore micro service.
- Remove the previous image.
- Save the Entity.

Originally the code looked like this:

{% gist codingtim/737ca0f5b3bec43d68b7 %}

The Entity and ImageEntity are POJOs with simple getters and setters for their properties.

My biggest problem with this code is that the state of the Entity is modified in multiple places. This leads to code that is hard to figure out. To improve this I treat the ImageService and underlying ImageStoreGateway as a standalone component and refactor to:

{% gist codingtim/c033a2625a3d461ede11 %}

This way the ImageService no longer needs to know about Entities and more importantly doesn't alter the state of an Entity. ImageService could be moved to its own package to better show that it's a standalone component. To improve this even further I get rid of the setters:

{% gist codingtim/356436e0ae450e4ee077 %}

Almost happy, but I don't believe the update method should be responsible for removing the old Image. In order to get a separate class to handle this I would use an observer. This observer receives the original and the updated Entity and can decide if any action is required. To ease having a reference to the original Entity we could make the class immutable. By making the Entity class immutable the update method will return a new Entity and the Entity we retrieved from the repository will still have the old values.

{% gist codingtim/90ba3c4e7d429717f704 %}
