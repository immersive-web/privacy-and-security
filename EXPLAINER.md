# Explainer - AR Privacy & Security on the Web


## Scope

This explainer outlines security and privacy **considerations**, including specific threat vectors, and possible **mitigation** options for user agents that wish to implement augmented reality on the web. The purpose of this explainer is to inform the development of web specifications related to augmented reality.

It is not the purpose of this explainer to prescribe a specific solution. This document is not a specification, nor is the intent to specify requirements for specifications. All text in this explainer is intended as informative only, and should not be treated as normative.


## Background

Augmented reality systems typically use one or more sensors to infer information about the real world, and may then present information about the real world to the user. In such systems there may be a wide range of input sensor types used, and a range of real-world data generated. Further, sensors (e.g. the camera) may be accessed and used to render information in context for the user.


## Accessing Real-World Data

A specification for augmented reality might allow the site to access information about the real world environment around the user. There are a variety of types of such information, ranging from real-world geometry (e.g. planes or point clouds representing actual objects in the user's space), to the ability to identify and track specific objects in the user's space. Different data types have different considerations, potential threat vectors and mitigations.


### Real-World Geometry

Geometry representing objects in the real world can be generated using one or more sensors, including an RGB camera, or a depth camera. Such data today may be as imprecise as ~5cm but this is expected to become more precise over time as sensor fidelity, algorithms, and the ability to combine sensor inputs evolve. However, such geometry data can be generated today from just camera data at varying levels of permission.


#### Threat Vectors

There are wide range of threat vectors relating to site access to real-world geometry, for example:


###### User PII



*   Users may be identified by learning the geometry of their face, or through gait analysis
*   In cases where the user is not visible to the AR system, identifying a user's associates may allow sites to identify the user
*   Credit card information may be pulled by analyzing embossed card geometry


###### Fingerprinting



*   Room geometry may be used to identify when two sessions are occuring in the same space


###### User Location



*   Real-world geometry may indicate a specific location. For example, the user might be near a recognizable object (e.g. the Eiffel Tower) or some place with unique geometry characteristics that the site can identify.


###### User Profiling



*   Estimating the size of the user's house, to estimate user income.
*   User ergonomics may be inferred. For example, users will typically hold phones at the same height, in the same location, and user height may be inferred by identifying the ground plane and the device's relative position to it.
*   Determine what businesses or apartments the user has visited, if enough real-world geometry is available to create a continuous map segment between the user's current location and other locations.
*   In combination with other technology such as eye tracking, could allow inference of what the user is interested in or attracted to.


###### User Physical Safety

*   The occlusion of real-world objects pose potential threats to users. If a user cannot see a stop sign they might get into a traffic accident; if the user cannot see a chair on the floor they might trip over it. On devices where a site can directly obscure a user's view of the real world, access to real-world geometry may allow sites to identify and occlude objects related to user safety. For example, a site might use real-world geometry to identify the octagonal shape of a stop sign and render an object to occlude it.


##### Precision of Real-World Geometry

Even a little real-world geometry data poses a potential security risk. For example, exposing a simple API such as hit-testing real-world objects with a ray can be used quickly to identify the floor plane and user height, and with enough data points can be used to build a point cloud in space from which real-world geometry can be inferred.

Further, even simplified geometry (such as bounding boxes) can provide information about the user. For example, bounding boxes of a certain size and placement can indicate that the user is in a kitchen, a bedroom or outdoors.


##### Historical Real-World Geometry

An additional privacy concern with world geometry data applies to AR subsystems that continuously improve their map of world geometry during system use, so as to not require the user to manually rescan their environment again each time they enter a new app or return to an existing app.

On such systems, some geometry data that is available from the subsystem may have been captured outside of a user's current user agent session, perhaps in previous device sessions. Processed world geometry can thus create novel privacy concerns that go beyond raw camera data, as they can pull in _historical_ perception of the environment.

This may be in conflict with user expectations for privacy and security. When a user enables their webcam, it is generally understood that the page can see the user's surroundings in the present, but any objects put away prior to the webcam session will stay private. In contrast, an AR subsystem that captures world geometry during general usage might still have information about objects or parts of the world that are not presently visible. It is unclear whether users will understand that such historical data exists and whether such data is available to the site.

Further, on such systems the geometry generated during a browsing session may persist within the subsystem beyond the browsing session, potentially allowing other applications or websites to access that data. This is in conflict with user expectations for privacy and security, where it is generally expected that data generated on a web page is not available to other sites or applications.


#### Possible Mitigations for Real-World Geometry Considerations


##### Throttling and Precision

Geometry precision is typically necessary to do even basic AR tasks. For example, tracking an object's location in a room, or measuring an object, requires that the data be reasonably accurate.

Certain threat vectors can be mitigated by only providing a rough estimate of real-world geometry to a site. For example, while detailed geometry could identify credit card numbers or faces, this is unlikely if the site only has access to two or three polygons representing the card or the face.

However, it's unclear how much value lowering precision or introducing error would have. Scrambling or de-precising geometry data could potentially be worked around by restarting the session several times; and analysis of the differences in errors may allow inference of what the actual 3D data is.

Further, it's not clear what value data throttling would bring. Even a small amount of data can pose a security risk, and without a hard limit on data access, over time a site could gather a large amount of geometry data. For example, if geometry gathering was limited to a user action, sites could design experiences (e.g. games) which solicit frequent user actions to get as much geometry as possible.

In both cases, providing low-resolution or inaccurate data might trade off perceived safety for app quality, but but given the low amount of data necessary to pose a risk it is possible that such an approach would hobble the API (degrade the user experience) without actually making it safer.

A user agent might choose to allow users to explicitly choose the data fidelity available to a site, possibly with a visual indication of what data resolution is being provided, so that the user can estimate both the threats to their privacy as well as the impact on their user experience.

_<span style="color:#24292e;">TODO: Provide examples<span style="color:#24292e;"> of decimated, lower-resolution data sets so the reader can get a feel for what information is actually available at various geometry levels.</span></span>_


##### Filtering
On systems where historical geometry is available beyond the scope of the current session, user agents may wish to filter what data is available to sites to only include data present in the user's current browsing session.

For example, a user agent might limit the distance and location of data that is available, to only that _near_ the user. This approach has limitations and might still expose data that is not visible to the user (for example, geometry for parts of the world directly behind a wall in the user's space). This approach may also limit AR use cases (for example, in a large room or outdoors, some geometry might be beyond the allowable distance).

Similarly, a user agent might limit the scope of data to only that _visible_ in the user's view, or data about geometry that has actually been seen by the user in the current session. For example, a hit testing API might limit results to those visible in the current view by only providing the closest hit-test result (thus not including anything that is occluded) and by restricting rays to those within the user's viewport, originating from the device pose. However, this approach may not guarantee that historical data is excluded. For example, if there is not yet any geometry information known about the visible user's space, then hit tests may return results from known, adjacent areas without a way of disambiguating whether those results are visible or not.

The approach that a user agent takes will likely depend upon the mechanisms offered by the underlying AR system. For example, a subsystem may have mechanisms for filtering real-world geometry to that seen within the current session, or may have its own context management system for managing access to previously mapped areas such as rooms.


##### Clearing Data

On systems where historical geometry is available between sessions, user agents may wish to clear or delete real-world geometry to avoid giving sites access to historical data, or giving other applications access to data generated during the browsing session.

For example, at session start the user agent might clear any historical data present on the system, only using data generated by the sensors in the current user's context. This approach might require the user to spend time scanning their environment before the AR experience can execute.

Similarly, the user agent might clear any real-world geometry data generated by the browsing session when the session ends. This may be complicated if the underlying AR subsystem has has merged some of this geometry with its larger real world understanding; for example, merging newly discovered floor planes into a larger plane that includes historical data.

The approach that a user agent takes will likely depend upon the mechanisms offered by the underlying AR system. For example, a subsystem may have mechanisms for sandboxing real-world geometry between app sessions.

### Object or Image Identification

Another type of real-world data access is the ability to detect objects or images in the scene. Such identification might include 3D object detection (e.g. identifying and tracking a 'dining room table') or 2D planar image detection (e.g. identifying and tracking a poster of [Rick Astley](https://www.youtube.com/watch?v=dQw4w9WgXcQ)).

#### Threat Vectors

Depending upon how the API is structured, the user may not know what objects the site is looking for. For example, if an API accepts a set of reference images and attempts to match them in the real world, the user likely would not know what those references images are.

Using such APIs, the site could gain information about what is present in the user's environment, such as:

*   Detecting that there is high-denomination money
*   Detecting valuable furniture or electronics
*   Detecting publicly visible objects or images such as storefront signs

This capability exposes several threat vectors including:


###### Location Identification
*   Storefront signs, or similarly known images, might allow a site to determine a user's location
*   Publicly visible QR or barcodes could allow a site to track user location
*   A unique configuration of a set of known objects (e.g. picnic tables) might allow a site to recognize the user's location (by knowing the configuration of picnic tables in parks)


###### User Profiling
*   Identifying expensive items, particularly in conjunction with location, could allow a bad actor to determine targets for real-world burglary
*   Identifying expensive items or currency could allow demographic-based ad targeting


###### Compromising Information 
*   Identifying embarrassing imagery in the user's space might allow a site to embarrass the user

###### Unique Codes
*   Detecting QR or bar codes in the scene could allow a site to determine the presence of specific objects, or user's location


###### Obscuring Real-World Data
*   Detecting and obscuring certain objects in the user's field of view could pose safety issues for the user. For example, a site might detect stop signs and obscure them.

#### Possible Mitigations for Object Detection

##### Throttling

Limiting the number of objects or images that the site can identify in a given session could help protect the user from broad-ranging attacks.

##### User-Visible Queries

If the user has the ability to visualize what the site is looking for (e.g. the ability to preview reference images), this could help the user ensure that only relevant information to their task is being searched for.


##### Limited Location or Tracking

A user agent may choose to prevent a site from accessing specific locations, or the ability to track identified objects. Such limitations could mitigate certain threat vectors (such as identifying the exact configuration of objects). However, even a declarative method (e.g. "place this 3D model on a $100 bill") could allow the site to know that certain objects are in the scene (e.g. when the site knows the model is placed).


##### Composition Rules

A user agent or underlying systems could establish rules for how imagery is composited into the user's field of view, to prevent obscuring important objects or critical parts of the user's vision, to ensure user safety.


## Permissions and User Consent

### Perception of Camera Access

#### Threat Vectors
...
#### Possible Mitigations
...

### Permissions

#### Considerations
...

#### Possible Mitigations
...




