---
layout: post
title: "HTML5 Geolocation: Improving GPS Accuracy"
author: Jordan MacDonald
date: 2012-03-01T18:41:00
comments: true
categories: [JavaScript, HTML5, Geolocation]
---
The new geolocation capabilites in the HTML5 spec are great, but it's easy to get wildly inaccurate results using the `getCurrentPosition` method. The main problem with this method is that it doesn't interface with the GPS until you call it, and only leaves it active for a split second. If you've ever used a pure, non-assisted GPS (like the regular handheld Garmin units), you're aware that it can take up to a few minutes for the GPS to find a satellite and start working. Smartphones help alleviate this initial delay by using _assisted GPS_, but that still doesn't provide instantaneous access to accurate location services. All told, to get the best results, the GPS should be running _before_ you make request location information from it.

The HTML5 geolocation spec provides another method, aptly named `watchPosition`, which continuously tracks your location once called. By calling this method as soon as your app starts, the GPS hardware will activate immediately and keep running, and it'll fire callbacks whenever the device's location has changed. To handle this somewhat quirky approach of calling this method before you actually need location information, declare and use a toggle variable to help you determine when you actually need to do something with the data in the callback.

Let's do just that:

``` coffeescript
positions = []
tracking_position = false
navigator.geolocation.watchPosition
  # Success callback
  (position) ->
    positions.push positions if tracking_position
  # Error callback
  (error) ->
    console.log 'Uh oh, an error occurred.'
  # Options
  {enableHighAccuracy: true}
```

Using this approach improves the accuracy of the GPS dramatically compared to the on-the-spot `getCurrentPosition` method, and is indispensable if you need accurate GPS data from the browser.
