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

Similarly, the user agent might clear any real-world geometry data generated by the browsing session when the session ends. This may be complicated or impossible if the underlying AR subsystem has has merged some of this geometry with its larger real world understanding; for example, merging newly discovered floor planes into a larger plane that includes historical data.

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

Augmented reality systems typically enhance some part of the user's field of view, and may either render the camera behind those enhancements (e.g. on a smartphone) or use the camera for visual realism (e.g. refraction). Even if the site cannot access camera data, the _perception_ that the site has access to camera data is a consideration.

#### Threat Vectors

User perception of camera access is an actionable threat vector, even if the site cannot access camera data. Consider the following scenario:

1.  A malicious site could create an AR session that displays camera (and nothing else);
1.  A smartphone user could visit that site, and be presented with a front-camera view, potentially capturing a compromising situation;
1.  The user would likely close the camera view, quickly;
1.  The site could then say "Thank you for that picture! We'll add to the public gallery. Please pay our $10 membership fee to control who can see the picture."

In this case, even if the site does not have access to camera data, it is likely that some percentage of users will be fooled into paying the membership fee. More generally, these threat vectors could be similar to malware which falsely claims that the user agent is compromised.

Such threat vectors could be a concern even on devices without cameras. Consider the following scenario:

1.  The user is using an optically see-through device;
1.  A malicious 2D web page - without access to camera data - recognizes that it is on an optically see-through device (e.g. detecting through CSS that it is on an additive display);
1.  The page renders a camcorder viewfinder with a blinking REC indicator in the corner (e.g. on an additive display, the viewfinder could be filled with black pixels, and would thus show the real world). This would make it appear that the web page can record real world imagery, even though it can't.

If not managed properly, the perception of unauthorized camera access could result in a negative perception of the user agent. For example, "Did this browser just give a site access to my camera without permission!?"

#### Possible Mitigations

If the site does not have access to the camera, ideally the user agent will communicate that fact explicitly to the user to mitigate the false perception. Other UX approaches might achieve the same outcome.

Given the threat vectors associated with the perception of camera access, user agents may wish to be explicit about whether a site does or does not have access to camera data (independent of other sensor data)


### Permissions

#### Considerations

It is likely that the user agent will wish to gain user consent, or at least notify the user about what data is being accessed, when entering an web-based augmented reality session. In such sessions, it is possible that the site will have access to one or more sensors and/or one or more types of real world data.

Traditional web permissions could be used to solicit user consent. However, this approach has several considerations:

##### Over-prompting and fatigue

Users that get prompted for permission(s) every time they enter an AR experience could be trained to accept _all_ types of permissions without reading them. For AR in particular, a site may be able to train a user to accept an AR permission and then prompt for (and gain) permanent camera permission without the user reading the prompt or understanding that the camera permission is not related and/or restricted to the AR capabilities.

##### Ambiguity, Complexity

Web permissions may make it difficult to accurately describe what is happening. AR may expose a wide range of data to the site, and it's difficult to describe that data access in a manner that is clear to the user. In some ways, _incomprehensible_ permissions are worse than no permission, and existing permission user experiences are often limited in how much text it can display.

Further, the user may not connect a web permission with the AR session. The current permissions user experience may confuse users who might approve or deny it without realizing that it is a gatekeeper to a camera-based AR experience (i.e. they think it's just another permission).

##### Data access beyond the life of the session

There are [threat vectors](https://github.com/immersive-web/privacy-and-security/issues/6) associated with a site having access to resources after the AR session ends, particularly if those resources are general permissions (e.g. camera), but even the [perception of camera access is a threat vector](https://github.com/immersive-web/privacy-and-security/issues/3), meaning that a long-running AR permission is a potential danger. Web and native permission models have generally included some amount of persistence (either permanent or time-limited) to reduce friction and over-prompting.

Further, it is expected that augmented reality experiences will be desirable for users, and sites can use the incentive of an augmented reality experience to gain consent. Because of this, malicious sites could gain long-running AR permissions for one safe context, and then solicit those users to visit malicious experiences later (where the user may not realize that they've already given permission to the site).

##### Consent vs. Notification

Some augmented reality specifications may support AR sessions where no data is shared with the site. In those situations, user agents may still wish to notify the user what is happening (e.g. a declarative AR API where the site cannot access the [camera](https://github.com/immersive-web/privacy-and-security/issues/3), and the user agent wants to be clear on that point). The existing web permissions model does not support this.

##### User Control

With augmented reality, a user may wish to configure what data is available to the site within a session. For example, the user may wish to allow some level of geometry access, but block the ability to access the camera or the ability to identify specific objects. This type of configuration within the scope of a session is not available with web permissions.

#### Possible Mitigations

There are several approaches that could mitigate the concerns with using web-based permissions:

##### Time-limited permissions

To address concerns about the long-running nature of web permissions, a user agent may wish to time-limit permissions. It is not clear that giving users explicit control would solve this problem, but permissions could be session-based by default. However, this approach may not adequately inform the user as to the connection between the AR session and the granted permission, and may additionally lead to over-prompting and permission fatigue.

##### Augmented Reality Mode

Instead of permissions, an alternative approach could be an explicit "AR Mode" which may ask for user consent before entering, with consent lasting only as long as the page is in this mode (i.e., session).

Such a mode would have the following advantages:

1.  _Flexibility_. The user agent could either ask for consent, or notify the user, or both. A user agent could further choose a UX for this mode allowing presentation of detailed information to the user without the constraints of traditional permissions user experience. This approach also gives the most opportunity to customize the experience across a variety of form factors, some of which may have different data sharing or user interface requirements than others.
1.  _Clear Scope_. Similar to Full Screen and VR presentation modes, this mode may be clearer to users than permissions that they are entering a specific mode, and there could be clear instructions on how to leave that mode. The origin that has data access during the mode can also be clearly indicated, and the user interface could be clear to the user that they're entering a specific experience, not just giving the site access to some data.
1.  _Enforcement_. A specific AR mode could enforce data access directly connected to what the user was notified of, and what user consent was provided. This may be less clearly defined with a permission that could (in theory) be requested independent of the actual session creation.

