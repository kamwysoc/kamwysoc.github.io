---
layout: post
title: 'Swift: Decode different objects from JSON array'
author: kwysocki
date: 2023-07-03 20:00
category: blog
star: true
tag: 
- iOS 
- swift 
- JSON
- decoding
- maping
---

# Decoding different objects from JSON array in Swift

## Problem: Decode objects from an array with different objects 

![title image >](/assets/posts/decode-different-objects/different_objects.png)

```swift
struct Car: Decodable {
    let manufacturer: String
    let model: String
}

struct Person: Decodable {
    let name: String
    let lastName: String
}

let json = """
{
    "information_array": [
        {
            "manufacturer": "Audi",
            "model": "A3"
        },
        {
            "name": "John",
            "lastName": "Doe"
        },
        {
            "weatherType": "windy",
            "degrees": "10",
            "degreesType": "celcius"
        }
    ]
}
"""
```

## Solution

```swift
struct OptionalDecodable<T: Decodable>: Decodable {
    let base: T?

    init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        self.base = try? container.decode(T.self)
    }
}

extension Decoder {
    func decodeObject<T: Decodable>() throws -> T {
        let container = try singleValueContainer()
        guard let object = try container.decode([OptionalDecodable<T>].self).compactMap({ $0.base }).first else {
            let context = DecodingError.Context(codingPath: container.codingPath,
                                                debugDescription: "Value for type \(type(of: T.self)) not found")
            throw DecodingError.valueNotFound(T.self, context)
        }
        return object
    }
    
    func decodeObjects<T: Decodable>() throws -> [T] {
        let container = try singleValueContainer()
        let objects = try container.decode([OptionalDecodable<T>].self).compactMap({ $0.base })
        return objects
    }
}

struct ContainerObject: Decodable {
    let car: Car
    let person: Person

    enum CodingKeys: String, CodingKey {
        case infoArray = "information_array"
    }

    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        let arrayDecoder = try container.superDecoder(forKey: .infoArray)
        self.car = try arrayDecoder.decodeObject()
        self.person = try arrayDecoder.decodeObject()
    }
}
```