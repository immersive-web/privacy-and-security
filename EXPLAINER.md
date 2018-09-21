# Explainer - AR Privacy & Security on the Web


## Scope

This explainer outlines security and privacy **considerations**, including specific threat vectors, and possible **mitigation** options for user agents that wish to implement augmented reality on the web. The purpose of this explainer is to inform the development of web specifications related to augmented reality.

It is not the purpose of this explainer to prescribe a specific solution. This document is not a specification, nor is the intent to specify requirements for specifications. All text in this explainer is intended as informative only, and should not be treated as normative.


## Background

Augmented reality systems typically use one or more sensors to infer information about the real world, and may then present information about the real world to the user. In such systems there may be a wide range of input sensor types used, and a range of real-world data generated. Further, other sensors (e.g. the camera) may be accessed and used to render information in context for the user.


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



*   Estimating the size of the user's house, to estimate user income
*   User ergonomics may be inferred. For example, users will typically hold phones at the same height, in the same location, and user height may be inferred by identifying the ground plane and the device's relative position to it.
*   Determine what businesses or apartments the user has visited, if enough real-world geometry is available to create a continuous map segment between the user's current location and other locations


##### Precision of Real-World Geometry

Even a little real-world geometry data poses a potential security risk. For example, exposing a simple API such as hit-testing real-world objects with a ray can be used quickly to identify the floor plane and user height, and with enough data points can be used to build a point cloud in space from which real-world geometry can be inferred.

Further, even simplified geometry (such as bounding boxes) can provide information about the user. For example, bounding boxes of a certain size and placement can indicate that the user is in a kitchen, a bedroom or outdoors.


##### Historical Real-World Geometry

<span style="color:#24292e;">An additional privacy concern with world geometry data applies to AR subsystems that continuously improve their map of world geometry during system use, so as to not require the user to manually rescan their environment again each time they enter a new app or return to an existing app.</span>

<span style="color:#24292e;">On such systems, some geometry data that is available from the subsystem may have been captured outside of a user's current user agent session, perhaps in previous device sessions. Processed world geometry can thus create novel privacy concerns that go beyond raw camera data, as they can pull in _<span style="color:#24292e;">historical<span style="color:#24292e;"> perception of the environment.</span></span>_</span>

<span style="color:#24292e;">This is in conflict with user expectations for privacy and security. When a user enables their webcam, it is generally understood that the page can see the user's surroundings in the present, but any objects put away prior to the webcam session will stay private. In contrast, an AR subsystem that captures world geometry during general usage might still have information about objects that are not presently visible. It is unclear whether users will understand that such historical data exists and whether such data is available to the site.</span>

<span style="color:#24292e;">Further, on such systems the geometry generated during a browsing session may persist within the subsystem beyond the browsing session, potentially allowing other applications or websites to access that data. This is in conflict with user expectations for privacy and security, where it is generally expected that data generated on a web page is not available to other sites or applications.</span>


#### Possible Mitigations for Real-World Geometry Considerations


##### Throttling and Precision

Geometry precision is typically necessary to do even basic AR tasks. For example, tracking an object's location in a room, or measuring an object, requires that the data be reasonably accurate.

Certain threat vectors can be mitigated by only providing a rough estimate of real-world geometry to a site. For example, while detailed geometry could identify credit card numbers or faces, this is unlikely if the site only has access to two or three polygons representing the card or the face.

However, it's unclear how much value lowering precision or introducing error would have. Scrambling or de-precising geometry data could potentially be worked around by restarting the session several times; and analysis of the differences in errors may allow inference of what the actual 3D data is.

Further, it's not clear what value data throttling would bring. Even a small amount of data can pose a security risk, and without a hard limit on data access, over time a site could gather a large amount of geometry data. For example, if geometry gathering was limited to a user action, sites could design experiences (e.g. games) which solicit frequent user actions to get as much geometry as possible.

In both cases, providing low-resolution or inaccurate data might trade off perceived safety for app quality, but but given the low amount of data necessary to pose a risk it is possible that such an approach would hobble the API (degrade the user experience) without actually making it safer.

_<span style="color:#24292e;">TODO: Provide examples<span style="color:#24292e;"> of decimated, lower-resolution data sets so the reader can get a feel for what information is actually available at various geometry levels.</span></span>_


##### Filtering
On systems where historical geometry is available beyond the scope of the current session, user agents may wish to filter what data is available to sites to only include data present in the user's current browsing session.

For example, a user agent might limit the distance and location of data that is available, to only that _near _the user. This approach has limitations and might still expose data that is not visible to the user (for example, geometry for objects directly behind a wall in the user's space). This approach may also limit AR use cases (for example, in a large room or outdoors, some geometry might be beyond the allowable distance).

Similarly, a user agent might limit the scope of data to only that _visible _in the user's view, or data about geometry that has actually been seen by the user in the current session. For example, a hit testing API might limit results to those visible in the current view by only providing the closest hit-test result (thus not including anything that is occluded) and by restricting rays to those within the user's viewport, originating from the device pose. However, this approach may not guarantee that historical data is excluded. For example, if there is not yet any geometry information known about the visible user's space, then hit tests may return results from known, adjacent areas without a way of disambiguating whether those results are visible or not.

<span style="color:#24292e;">The approach that a user agent takes will likely depend upon the mechanisms offered by the underlying AR system. For example, a subsystem may have mechanisms for filtering real-world geometry to that seen within the current session, or may have its own context management system for managing access to previously mapped areas such as rooms.</span>


##### Clearing Data

On systems where historical geometry is available between sessions, user agents may wish to clear or delete real-world geometry to avoid giving sites access to historical data, or giving other applications access to data generated during the browsing session.

For example, at session start the user agent might clear any historical data present on the system, only using data generated by the sensors in the current user's context. This approach might require the user to spend time scanning their environment before the AR experience can execute.

Similarly, the user agent might clear any real-world geometry data generated by the browsing session when the session ends. This may be complicated if the underlying AR subsystem has has merged some of this geometry with its larger real world understanding; for example, merging newly discovered floor planes into a larger plane that includes historical data.

<span style="color:#24292e;">The approach that a user agent takes will likely depend upon the mechanisms offered by the underlying AR system. For example, a subsystem may have mechanisms for sandboxing real-world geometry between app sessions.</span>

### Object or Image Identification

#### Threat Vectors
...
#### Possible Mitigations
...

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




