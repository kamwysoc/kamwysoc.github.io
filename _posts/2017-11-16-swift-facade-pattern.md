---
layout: post
title: 'Design Patterns with Swift: Facade pattern'
author: kwysocki
date: 2017-11-16 00:40
category: blog
start: false
tag:
- Swift
- iOS
- design patterns
- facade pattern
---

![facade image](/assets/posts/swift-facade-pattern/facade.jpg)

## About the pattern

Facade pattern is one of the Structural Patterns. The main aim of it is to hide the complexity of a system, class or logic and provide all functionalities behind a simple interface.
Commonly, Facade is implemented in a way that one class is related to other classes which represents a system logic. Please take a look at the diagram:

![diagram](/assets/posts/swift-facade-pattern/diagram.png)

As you can see, there is one class called `Facade` which separates the logic from `LogicA`, `LogicB`, `LogicC` classes. As a result our client only call the `Facade` class in order to execute some methods that are implemented in other classes.

## Implementation

Let's imagine a simple scenario. You have already created a great app called `Super-Photo`. One of the core features of your app is saving/converting assets/posts with `JPEG` or `PNG` extension. In order to do this you want to save `UIImage` representation in two ways. One is saving it as a `PNG` type, the second is saving it as a `JPEG` file type.

Fistly, in order to handle our image types and possible errors in our code - it will be nice to have two enums that will make our code cleaner and more readable.

```swift
enum ImageSaverError: Error {
    case couldNotCreateDestinationPath
    case couldNotCreateJPEGDataFromImage
    case couldNotCreatePNGDataFromImage
    case couldNotSaveImageInDestinationPath
}

enum ImageType {
    case png
    case jpeg(compressionQuality: CGFloat)
}
```

On the next step you will need to create a class that will handle a data providing for each photo extension:

```swift
class ImageDataProvider {
    func data(from image: UIImage, type: ImageType) throws -> Data {
        switch type {
        case .jpeg(let compressionQuality):
            return try jpegData(from: image, compressionQuality: compressionQuality)
        case .png:
            return try pngData(from: image)
        }
    }

    private func pngData(from image: UIImage) throws -> Data {
        guard let imageData = UIImagePNGRepresentation(image) else { throw ImageSaverError.couldNotCreateJPEGDataFromImage }
        return imageData
    }

    private func jpegData(from image: UIImage, compressionQuality: CGFloat) throws -> Data {
        guard let imageData = UIImageJPEGRepresentation(image, compressionQuality) else { throw ImageSaverError.couldNotCreatePNGDataFromImage }
        return imageData
    }
}
```

As you've noticed, our `ImageDataProvider` takes `image` and `type` parameters and creates the image data with proper extension `JPEG` or `PNG`.

The last step is to create class which is needed to save `UIImage`. So let's name it a `PathProvider`. 

```swift
class PathProvider {
    func createDestinationPath(fileName: String) throws -> URL {
        guard let path = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else {
            throw ImageSaverError.couldNotCreateDestinationPath
        }
        let destinationPath = path.appendingPathComponent("\(fileName)")
        return destinationPath
    }
}
```

Ok, so at the moment we have two classes with some logic inside. Now it's time to create a facade for it!

![let's do this](/assets/posts/swift-facade-pattern/do_this.gif)

Create a class called `ImageSaverFacade` :

```swift
class ImageSaverFacade {
    private let pathProvider = PathProvider()
    private let dataProvider = ImageDataProvider()

    func save(image: UIImage, type: ImageType, fileName: String, overwrite: Bool) throws {
        let destinationURL = try pathProvider.createDestinationPath(fileName: fileName)
        let imageData = try dataProvider.data(from: image, type: type)
        let writingOptions: Data.WritingOptions = overwrite ? (.atomic) : (.withoutOverwriting)
        try imageData.write(to: destinationURL, options: writingOptions)
    }
}
```

Our `ImageSaverFacade` class has two private objects of `PathProvider` and `ImageDataProvider` class. Because the client doesn't need to know anything about logic inside, the only thing which `ImageSaverFacade` exposes to a public is one method:

* `func save(image: UIImage, type: ImageType, fileName: String, overwrite: Bool) throws`

This method is the only thing that our client should care about.

Now let's move on to the facade usage part:

```swift
let imageSaver = ImageSaverFacade()
let image = UIImage(named: "my_image")!
do {
    try imageSaver.save(image: image, type: .png, fileName: "my_file_name", overwrite: true)
} catch  {
    //handle Error
}
// or
do {
    try imageSaver.save(image: image, type: .jpeg(compressionQuality: 1.0), fileName: "my_file_name", overwrite: false)
} catch  {
    //handle Error
}
```


## Conclusion

Please notice that our facade covers logic associated with `Data` providing and creating a valid `URL` for file destination. And because of that, it is super easy to save `UIImage` as `PNG` or `JPEG` using our `ImageSaverFacade`. Only thing to do is to pass the correct parameters to facade method.

Facade design pattern can be used in many cases. Facade creates for you a simple gateway to a complicated system. By using it you will definitely make your code simpler to understand and read.

This post was primarly posted on my company [blog](https://brightinventions.pl/blog/swift-facade-pattern/)