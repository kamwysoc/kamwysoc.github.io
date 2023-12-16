---
title: 'What is new in Swift 4.2 - my summary'
layout: post
author: kwysocki
date: 2018-06-06 20:00
category: blog
star: false
headerImage: true
tag: 
- iOS 
- Swift
- WWDC
---

![](/assets/posts/whatisnewinswift-mysummary/swift_image.png)

I just have watched the What's new in Swift from WWDC 2018 and I thought it is a great motivation to write a blog post about this talk and summarize what I learned.

And here are some new Swift 4.2 features that I really liked. 

Hope you will enjoy! 🤓

## SE-0194 Derived Collection of Enum Cases 

In case we need to print all available enum values, we had to create some helper variable that includes all enum cases. For example, a static array called `allCases`. A big drawback in that approach is that we need to remember to update the `allCases` array every time when we modify enum cases.

Swift 4.1 approach: 
```swift
enum CarType {
    case sedan
    case crossover
    case hothatch
    case muscle
    case miniVan

    static var allCases : = [.sedan, .crossover, .hothatch, .muscle, miniVan]
}
```

in Swift 4.2 we can work with `CaseIterable` protocol which does all the work for us! Please take a look at the below example:

```swift
// CaseIterable protocol gave us a `allCases` variable, which is an array of all cases in the Enum.

enum CarType : CaseIterable {
    case sedan
    case crossover
    case hothatch
    case muscle
    case miniVan

    //there is no need to add `allCases` variable. `CaseIterable` protocol do the job!
}
```

```swift
for type in CarType.allCases {
    print(type)
}
```

## Conditional Conformance

```swift
let arrayOfArrays = [[1,2],[3,4],[5,6]]

arrayOfArrays.contains([1,2]) // return false in Swift 4.1

arrayOfArrays.contains([1,2]) // now it returns True because of fact that the elements in the array conforms to Equatable protocol
```

It will work with `Optional`, `Dictionary` types as well.
The conditional conformance works in the same way with `Hashable`, `Encodable` and `Decodable` protocols.
So for example, because `Int` is `Hashable`, which means in that case that `Int?` is `Hashable` too, and as a result the `[Int?]` is `Hashable` as well!

```swift
let s: Set<[Int?]> = [[1, nil, 2], [3, 4], [5, nil, nil]]
s.contains([1,nil,2]) // returns true
```

## Bool toggle

```swift
var isTheWeatherNice : Bool = true
print(isTheWeatherNice) // prints true
//now it's starts to rain
isTheWeatherNice.toggle() // it will change the bool value.
print(isTheWeatherNice) // prints false
```

Small, but in my opinion -  very nice feature. I meet that extension for the first time while reading [that objc.io blog posts](https://www.objc.io/blog/2018/01/16/toggle-extension-on-bool/).

Now it's built into Swift 4.2. 🎉

## Hashable protocol 

```swift
protocol Hashable {
    func hash(into hasher: inout Hasher)
}
```

In Swift 4.2 we don't have to provide custom algorithms for `hashValue`. Now swift handles a hash method quality with run performance. 
Important thing is that the `hashValue` use the random per-process seed which is created at the every app starts.

```swift
struct City: Hashable {
    let name : String
    let state : String
    let population : String
}
extension City : Hashable {
    func hash(into hasher: inout Hasher) {
        name.hash(into: &hasher)
        state.hash(into: &hasher)
    }
}
let warsaw = City(name : "Warsaw", state: "Mazowieckie")
print(warsaw.hashValue) // will print hash value, using the Swift algorithms from hash function.
```

⚠️
In that approach, you should change the code that relates to the `hashValue` as a constant. In every application run, the hash value will be different.
⚠️


## SE-0202 Random Unification

Swift 4.1 approach:

```swift
let randomIntFrom1to10 = 1 + (arc4random() % 10) // return random number is the 1...10
```

But in Swift 4.2 there is no need to use `arc4random()` anymore. 🎉

```swift
let randomIntFrom0To20 = Int.random(in: 0 ..< 20)
let randomFloat = Float.random(in: 0 ..< 1)
```

Super cool thing is that we can get a random value from Collection types like `Array` or `Dictionary`.

```swift
let names = ["John", "Paul", "Peter", "Tim"]
names.randomElement()! 

let playerNumberToName : [Int: String] = [9: "Lewandowski", 7: "Ronaldo"]
playerNumberToName.randomElement()! 
```
As you might notice, the `randomElement` function returns an Optional, because of the case where we call this function on the empty collection.

```swift
let emptyCollection : [String] = []
emptyCollection.randomElement() // retuns nil
```

Another new function are `shuffle` or `shuffled` functions.

```swift
let names = ["John", "Paul", "Peter", "Tim"]
let shuffledNames = names.shuffled() // returns an array of names in shuffled order.
```

## Conclusion

It would be great to use those features in stable versions. My impressions from Xcode 10(beta) and Swift 4.2 was pretty amazing. I highly recommend you to watch What's new in Swift talk from WWDC 2018

Below you can find a link to a GitHub gist with all features described above.

[https://gist.github.com/kamwysoc/e9322c84fd4fa051cb747ec08193dc0d](https://gist.github.com/kamwysoc/e9322c84fd4fa051cb747ec08193dc0d)

#### Source
* [https://developer.apple.com/videos/play/wwdc2018/401/](https://developer.apple.com/videos/play/wwdc2018/401/)

* [https://swift.org/documentation/](https://swift.org/documentation/)
