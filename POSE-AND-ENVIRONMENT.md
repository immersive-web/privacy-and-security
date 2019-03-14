# Pose and Environment Data

## Background

Immersive Web APIs may provide both pose data (position/orientation) and data about the user's environment and context (data in a 'reference space') to site developers so that they can render 3D scenes for VR and AR. This document outlines potential threat vectors and mitigations for such data.

### Data Considered

_Floor Height:_ Through a reference space a developer may access the height of the user's viewpoint from the floor. On some systems this may be an accurate representation of height; on others it may be a user-set value or (when not provided) be a default value set by the client or user agent.

_Boundaries:_ Through a reference space a developer may access the boundary of the user's allowed real-world walking space (for example, a safety boundary). This may be represented as an arbitrary geometric outline at real-world scale relative to an arbitrary origin.

_Pose:_ Depending upon what reference space type is used, a developer may access the orientation and possibly position of a viewer, which represents either the user's head or device. This is often in real-world scale relative to an arbitrary origin. Some reference spaces may limit pose data, for example by restricting position to within a boundary, restricting viewing angle to within less than a full 360 degrees, or locking position to a fixed point.

Some frames of reference may provide multiple poses - for example, an HMD may provide a pose for each eye (to enable stereoscopic rendering), other devices may support multiple displays (such as a bank of monitors). In this case, even if no absolute position is provided the relative position between the provided poses may vary and provide additional information about the user (e.g. interpupillary distance, or monitor locations).

### Device Support

This document assumes that any device could support all types of data considered:

*   Some smartphone devices today provide device orientation sensors and may be able to infer floor height through camera data using SLAM libraries such as ARCore and ARKit;
*   Some laptops today include device orientation sensors and cameras, giving them similar capabilities to smartphones;
*   VR headsets today may provide floor height, pose data, and immersive bounds;
*   Fixed-position devices may use a user-facing camera to determine user location, head and eye orientation relative to the screen.

While it is assumed that all devices could potentially get all types of data, only some devices are likely to obtain multiple simultaneous poses (e.g. an HMD may get a pose for each eye.) This is assumed to depend upon the reference space type and the device form factor.

## Threat Vectors

### Gaze Tracking

_Exposed by: Orientation. Threat increases with full pose data (both orientation and position)._

Sites may be able to determine what the user is looking at through analysis of pose orientation. While pose data is not intended to provide precise eye tracking, in some contexts the direction of a viewer may provide an approximation of what the user is looking at.

For example, while using an immersive headset the viewer may adjust their head orientation as they look at different 2D parts of a web page, at objects outside the page, or at an inline 3D elements on the web page (where the viewer may adjust their gaze to look at the 3D element from different directions). In such situations the web page may be able to track user interest in parts of the web page, or interest in their surroundings.

Note: Specific to browsing a 2D web page, gaze tracking may be limited to the contents of the site itself, as the site may have knowledge of its own contents. Outside the site content, gaze tracking may only be a threat if the site also has an understanding of what is outside the web page (e.g. access to a camera feed showing the real world, or access to real-world geometry).


### Reading Inputs (or "Input Sniffing")

_Exposed by: Orientation. Threat increases with full pose data (both orientation and position)._

Mobile device orientation may [be used](https://arxiv.org/pdf/1602.04115.pdf) to reliably determine a user's input actions (e.g. scrolling, tapping). Similar threat vectors may exist where pose is affected by device orientation, and where device orientation may be affected by user input. Examples of input devices that may affect device orientation include physical keyboards (e.g. on a laptop), touch-screen inputs (e.g. a mobile touch keyboard or PIN pad), or virtual keyboards (e.g. in an HMD).

Generally speaking, this threat vector may be a concern in situations where a site or origin has access to pose data, while input is being entered elsewhere, and that input is not intended to be available to the site or origin. For example, when:



*   Pose data is available to a website that is occluded. This includes the website being behind a virtual keyboard, behind a PIN pad, or in a background tab;
*   The site is fully visible but not in focus, as in an open, non-focused desktop window;
*   Pose data is available to an origin on a website while another origin on the same website is soliciting input.


### User Profiling

_Exposed by: Pose, Floor Height, or Boundaries._

Floor height, or pose data, may be used to profile user physical characteristics such as height or even facial characteristics. Pose data may be used to profile a user's activities - for example, whether they are walking, running, or stationary. The user's gait, as exposed through pose data, might even be used to profile disabilities or other characteristics. On devices where a pose is provided for each eye, the interpupillary distance (IPD) may be used to estimate a user's head size. For example, this may be used to determine whether or not the user is a child.

The boundaries of a user's session may be used to infer user characteristics or context. For example, a large boundary may suggest that the user is affluent and can afford a large room. Similarly, the specific shape of a boundary may be used to determine what room the user is in, or whether they are outside.


### User Fingerprinting

_Exposed by: Pose, Floor Height, or Immersive Bounds._

User fingerprinting is the ability to establish that the user is the same between two sessions, based on unique characteristics of their configuration. 

Session boundaries - particularly when established using physical placement of sensors - may be sufficiently unique to allow user fingerprinting.

A user's physical characteristics may affect pose data and may allow fingerprinting (for example, by analyzing a user's gait). A user's floor height (whether set manually or algorithmically) may be unique. In sessions that provide multiple poses, the relative difference in position and orientation between those poses may allow fingerprinting (for example, on devices that allow configuration of interpupillary distance.)

Device orientation sensors have the potential to enable fingerprinting, particularly if combined with other browser fingerprinting signals [[1](https://arxiv.org/pdf/1605.08763.pdf)] [[2](https://arxiv.org/pdf/1503.01874.pdf)]. If pose data is directly based upon such orientation sensors then that pose data may enable fingerprinting. While the research was based on phone sensors, the same issues may apply to headsets and other form factors.


### User Location

_Exposed by: Pose Data gathered over a sufficiently large region._

Particularly if the session is non-stationary and unbounded, a site may be able to infer user location by tracking the user's position w.r.t. to an arbitrary unknown origin and correlating with known pathways or roads.

For example, on a university campus tour, the user's path may have high correlation to the pathways of that campus and thus the site may be able to infer (a) that the user is on the campus and (b) the user's exact location on that campus, even if the user does not provide additional input.

As another example, [research suggests](https://ieeexplore.ieee.org/document/8406600) that it may be possible to identify a user's location in major cities given less than 1000m of trajectory data when driving on roads.

### In Combination with Real-World Geometry or Geolocation

_Exposed by: Pose Data combined with real-world geometry understanding_

Pose data in combination with real-world geometry understanding or geolocation may compound certain threat vectors associated with both data types. For example, if pose data is used to perform gaze tracking outside the visible area of a web site, then real-world geometry or geolocation may allow a site to determine what real-world objects a user may be looking at. As another example, pose data in combination with real-world geometry may allow a site to infer user height, leading to the same threat vectors as Floor Height above.

## Possible Mitigations

### Focused and Visible

At an application level, ensuring that pose data is only available to an application while that application is in focus and visible can mitigate a range of threat vectors. For example, such a limitation prevents a site from accessing pose data influenced by inputs that are entered outside of the application, such as when the site is behind a PIN pad, when a 2D site is viewed within a 3D HMD environment, or when a keyboard on a laptop is used to enter a password in a different application or outside the browser.

Similarly, within browsers that allow multiple web applications to be open at the same time, ensuring that pose data is only available to a single application/frame that is in focus and visible can help mitigate a range of threat vectors. For example, this approach could prevent a site in a background tab from accessing foreground tab inputs, or prevent a non-focused foreground tab site from accessing input data solicited by a focused foreground tab site in another window.

In general, pose data should be limited to when it actually has potential value for the user and when providing such data could reasonably be expected by the user. For example, an inline experience does not need to be updated while the phone is in the user's pocket, so there is no value to giving the site pose data, only privacy costs. Specifically, pose data should generally not be provided when a device is locked or the display is asleep. This may prevent some use cases, such as audio-based experiences or user-desired path tracking, but those are the exception and should be handled with more explicit intent and/or consent mechanisms.

### Same-Origin, or Single-Origin Only

To address cross-origin threats, a user agent may wish to restrict access to pose data to only a single origin at a time. For example, restricting pose data to only the origin in focus may mitigate threat vectors related to cross-origin input sniffing. This may help mitigate both cross-origin iframe threats, as well as cross-origin DOM content that may be displayed in a fully immersive experience.

More generally, a user agent may wish to restrict access to pose data to only origins that are the same as the top-level document.

### Data Throttling

In theory, limiting pose input frequency may reduce a site's ability to infer gaze, input data, location, or perform user profiling. In practice however, such throttling would certainly affect the user experience and it is unlikely to provide additional security.

For example, even at 1Hz, the user's location and gaze information could likely be inferred. Similarly, while touch input data snooping was performed at frequencies as [low as 20Hz](https://arxiv.org/pdf/1602.04115.pdf), it is not known whether there is in practice a lower bound that prevents such detection. In either case display frame rates are likely to be higher than 20Hz and thus pose data at a higher frequency would be needed to maintain a reasonable user experience.

### Position Limiting

The user agent may wish to limit position information based upon which type of reference space the application has requested. For example, while a stationary reference space is intended to support applications that require limited or no movement, they still allow accurate positional data to be reported when available for user comfort. When using a stationary reference space, the user agent may wish to stop reporting pose data if the user moves beyond a limited radius from the origin.  This may help address threat vectors associated with location tracking for these types of reference spaces.

Note: Because positional changes may be reported for "stationary" reference spaces, policies and mitigations for such reference spaces must not assume that the position is fixed.

Note: The intent of position limiting is to limit data to the application, but care should be taken to ensure that the user experience is not negatively affected; it is generally undesirable to have a 'hard stop' at the position limits, as this may cause a disruptive user experience where the user is moving but the scene is not. The user agent may choose to transition to a different, non-application user interface when the viewer is outside the permitted position, and may choose to provide guides back to the permitted position.

### Rounding, Quantization or Fuzzing

Either rounding, quantization, or fuzzing (the introduction of slight, random errors) in data for floor height and session bounds may alleviate the threat of fingerprinting users using that data. For example, while a user's exact physical VR sensor locations may be unique when measured with 0.1cm precision, they may not be unique when measured at 10cm precision. Care should be taken to ensure that user safety and experience is not compromised when using this approach.

Sufficiently low precision in pose data may prevent sites from performing input detection, fingerprinting, and gaze tracking. Some AR modes on some devices may not require precise pose data. For example, one [study](https://arxiv.org/pdf/1605.08763.pdf) suggests that quantization of orientation sensor data to 6-degree increments may alleviate the threat of fingerprinting without significantly affecting tilt-based handheld smartphone experiences. Such quantization may be suitable for a handheld smartphone user viewing a 3D object inline on a web page.

However, it is unclear whether the same restricted level of precision will allow a meaningful experience when in an immersive mode, or when viewing inline content in an immersive headset; the suitability of low precision to effect useful experience will likely vary based upon the session mode and the device form factor.

### User Intent through Gaze

On systems where the user's gaze is known, making pose data only available to elements where the user is actively looking may help mitigate certain threat vectors. For example, a user agent may choose to only make pose data available for inline experiences when the user is actually looking at that inline element; this approach may prevent gaze tracking for elements outside the inline experience.

This approach alone may not mitigate threat vectors associated user profiling. It further may not mitigate threats related to reading inputs - for example, if a user is gazing towards a virtual keyboard and an (unfocused, background) site behind the keyboard, then the site may still be presented with pose data while the user entered data on the keyboard. Further, the act of limiting pose data to what the user is looking at may enable gaze tracking. For these reasons it is unclear why using the user's gaze to limit inputs would be preferable to the [Focused and Visible](#focused-and-visible) mitigation.

### Inline vs. Exclusive

A user agent may apply different privacy rules when the user is in an exclusive experience (e.g. fullscreen on mobile or an immersive mode on headsets) as opposed to an inline experience. For example, an exclusive experience may prevent gaze tracking for elements outside that exclusive experience by virtue of the fact that elements are not visible. Further, if the user agent controls the exclusivity of the experience, then the user agent may prevent other applications or sites from soliciting input.

### Position and Orientation vs. Orientation Only

A user agent may choose to apply different policies based on whether the site will access position and orientation data (a full pose) as opposed to only orientation data with a fixed position. Such an approach may alleviate the threats of identifying location and some user profiling. However, [research suggests](http://web.cs.ucdavis.edu/~hchen/paper/hotsec2011.pdf) that orientation alone is sufficient to read inputs, and thus it is unlikely that this approach would alleviate the threat of reading an input.

### Active User Engagement

A user agent may wish to prevent the availability of pose data unless the user has indicated that they are actively engaging with the session, page, and/or iframe that is requesting pose data.

### Visible Indicators

A user agent may wish to display a visual indicator to the user indicating that pose and/or other data is being shared with the site. 

### Consent

The user agent may require user consent before allowing the site to access pose or reference space data.

### Data Usage Validation

There may be mechanisms by which the user agent can validate that the use of pose data is legitimate. For example:

*   The user agent may wish to check GL matrices to ensure that they are tracking pose data, or ensure that there are visible changes that correlate with updates to pose data. Note that if the immersive API allows developers to change or ignore pose data, then this approach may not work.
*   The user agent may wish to only provide pose data for visible, non-obscured regions of a specific size or percentage of the window. This approach may help ensure that the user knows what is going on and prevent the use of invisible (e.g. 1x1) experiences.

### Generic Sensor Guidelines

Pose data, and in some cases reference space data, may be based upon sensors on the user's device. For such data, API specifications and user agents should follow the Generic Sensor API's security and privacy considerations unless there is a direct user-benefiting reason not to and appropriate mitigations to compensate are put in place:

  * [Secure Context](https://www.w3.org/TR/generic-sensor/#secure-context): Only expose APIs and data on secure origins
  * [Focused Area](https://www.w3.org/TR/generic-sensor/#focused-area): Only provide poses to the focused frame
  * [Same Origin](https://www.w3.org/TR/generic-sensor/#concepts-can-expose-sensor-readings): Only provide poses for frames that are same-origin to the top level document
  * [Visibility](https://www.w3.org/TR/generic-sensor/#visibility-state): Only provide poses on active documents whose visibility state is 'visible'
  * [Feature Policy](https://www.w3.org/TR/generic-sensor/#feature-policy): Pose and reference space data access should be aligned with feature policies for the underlying sensors that generate that data.
  * [User Consent](https://www.w3.org/TR/generic-sensor/#permissions): Where appropriate, require user consent, either through the Permissions API or through other means, before allowing the site to access data.

