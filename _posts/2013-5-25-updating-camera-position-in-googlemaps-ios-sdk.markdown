---
layout: post
title: Updating Camera Position in Google Maps iOS SDK
category: posts
---

Getting started with the Google Maps iOS Library was pretty straight
forward. Before Google made the documentation available,
attempting to figure out how to update the position of the camera was
not apparent to me.

The simplicity of this actual task is embarrasing, but for future
reference sake, here we go!

We are going to assume that you have a **GMSMapView** object and it has
already been set to a camera position. To update the location of the camera, we need, a
**GMSCameraUpdate** object. For this tutorial we will use the factory
method setTarget:zoom: to create said object.

{% highlight smalltalk %}
  GMSCameraUpdate *updatedCamera = [GMSCameraUpdate setTarget: zoom:];
{% endhighlight %}

**setTarget** expects a **CLLocationCoordinate2D** object and **zoom** a **CGFloat**.

With our newly created object, all we have to do is call the **animateWithCameraUpdate** message on our **GMSMapView** object (or the other messages that interact with the GMSCameraUpdate object). In our case the camera will animate to the new set of coordinates that we provide.

Let's move the camera position to the coordinates of the Barclays Center,
home of the Brooklyn Nets.

We will create a **CLLocationCoordinate2D** object and pass that along to
**setTarget**.

{% highlight smalltalk %}
  CLLocationCoordinate2D *coordinates = CLLocationCoordinate2DMake(40.683445,-73.976048);
{% endhighlight %}

**CLLocationCoordinate2DMake** takes a latitude and longitude and returns a
**CLLocationCoordinate2D** object.

Next we pass our coordinates to a **GMSCameraUpdate** object and bind that
to a variable.

{% highlight smalltalk %}
  GMSCameraUpdate *updatedCamera = [GMSCameraUpdate setTarget:coordinates zoom:10];
{% endhighlight %}

Lastly we call the **animateWithCameraUpdate** message on our **GMSMapView**
object.

{% highlight smalltalk %}
  [mapView animateWithCameraUpdate:cameraUpdate];
{% endhighlight %}

That's all there is to it! Now go forth and move thy cameras in Google
Maps!
