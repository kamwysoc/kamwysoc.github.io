---
title: 'Implementing a simple pedometer using Swift'
layout: post
author: kwysocki
date: 2017-12-08 17:00
category: blog
tag: 
- iOS 
- Swift
- CoreMotion
- pedometer
- tutorial
---

![footsteps image](/assets/posts/coremotion-pedometer-swift/footsteps.jpg)

Ok... but what the pedometer is? Here you have a wikipedia definition:

>A pedometer is a device, usually portable and electronic or electromechanical, that counts each step a person takes by detecting the motion of the person's hands or hips.

and YES, you can create your own pedometer using iOS framework called CoreMotion.

CoreMotion is a well-known iOS framework. As we could read in [docs](https://developer.apple.com/documentation/coremotion) it processes accelerometer, gyroscope, pedometer environment-related events.
In this post I want to focus on the pedometer events and how to handle them.

## Overview

### CMPedometer

In order to use CoreMotion pedometer, we need to take a closer look at `CMPedometer` class. It allows the user to retrieve some information about steps taken in the past, for example: How many steps I have done for last 3 days? Another usage of `CMPedometer` class is to get the live updates about steps taken already.

```swift

open func queryPedometerData(from start: Date, to end: Date, withHandler handler: @escaping CoreMotion.CMPedometerHandler)

open func startUpdates(from start: Date, withHandler handler: @escaping CoreMotion.CMPedometerHandler)

```

### CMPedometerData

Another class that should catch our attention is `CMPedometerData`. This class represents data that will be sent with every update in the above functions. It contains a lot of useful information like:

* `numberOfSteps: NSNumber?`
* `distance: NSNumber?`
* `currentPace: NSNumber?`
* `floorsAscended: NSNumber?`
* `floorsDescended: NSNumber?`


### CMMotionActivityManager

If we want to start counting steps, it will be good to know about what kind of activity our user is doing at the moment. Here with some help comes the `CMMotionActivityManager` class. Using th e instance of this class we are able to get updates about the user activity type. In order to do this we should call:

```swift
open func startActivityUpdates(to queue: OperationQueue, withHandler handler: @escaping CoreMotion.CMMotionActivityHandler)
```

and the result of that is getting updates with `CMMotionActivity` which represents  data for a single motion event update. This data is a pack of bool values:

* `stationary: Bool`
* `walking: Bool`
* `running: Bool`
* `automotive: Bool`
* `cycling: Bool`
* `unknown: Bool`


## Code step by step...

### 1. Add `NSMotionUsageDescription` to your `info.plist`

As we can read in [Apple docs](https://developer.apple.com/documentation/coremotion)

>Important
>An iOS app linked on or after iOS 10.0 must include usage description keys in its Info.plist file for the types of data it needs. Failure to include these keys will cause the app to crash. To >access motion and fitness data specifically, it must include NSMotionUsageDescription.

So add to your `info.plist` `NSMotionUsageDescription` key modifying plain file:

```xml
<key>NSMotionUsageDescription</key>
<string>In order to count steps I need an access to your pedometer</string>
```

or adding new key via Xcode

![info plist motion usage description](/assets/posts/coremotion-pedometer-swift/info-plist-motion-usage.png)

### 2. Create an `CMMotionActivityManager` and `CMPedometer` instances

```swift
private let activityManager = CMMotionActivityManager()
private let pedometer = CMPedometer()
```

### 3. Create a method for tracking activity events

```swift
private func startTrackingActivityType() {
  activityManager.startActivityUpdates(to: OperationQueue.main) {
      [weak self] (activity: CMMotionActivity?) in

      guard let activity = activity else { return }
      DispatchQueue.main.async {
          if activity.walking {
              self?.activityTypeLabel.text = "Walking"
          } else if activity.stationary {
              self?.activityTypeLabel.text = "Stationary"
          } else if activity.running {
              self?.activityTypeLabel.text = "Running"
          } else if activity.automotive {
              self?.activityTypeLabel.text = "Automotive"
          }
      }
  }
}
```

### 4. Create a method for steps counting updates

```swift
private func startCountingSteps() {
  pedometer.startUpdates(from: Date()) {
      [weak self] pedometerData, error in
      guard let pedometerData = pedometerData, error == nil else { return }

      DispatchQueue.main.async {
          self?.stepsCountLabel.text = pedometerData.numberOfSteps.stringValue
      }
  }
}
```

### 5. Start getting updates

```swift
private func startUpdating() {
  if CMMotionActivityManager.isActivityAvailable() {
      startTrackingActivityType()
  }

  if CMPedometer.isStepCountingAvailable() {
      startCountingSteps()
  }
}
```

![demo](/assets/posts/coremotion-pedometer-swift/steps-demo.gif)

## Conclusion

CoreMotion is a powerful framework and besides a pedometer it allows you to work with a plenty useful data from accelerometer and gyroscope as well
You can find an example project at [Github repository](https://github.com/bright/Pedometer-Swift)

Hope you like the post, feel free to share.


This post was primarly posted on my company [blog](https://brightinventions.pl/blog/coremotion-pedometer-swift/)