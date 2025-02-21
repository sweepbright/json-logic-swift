# jsonlogic-swift

[![CI Status](http://img.shields.io/travis/advantagefse/json-logic-swift.svg?style=flat)](https://travis-ci.org/advantagefse/json-logic-swift)
[![Version](https://img.shields.io/cocoapods/v/jsonlogic.svg?style=flat)](https://cocoapods.org/pods/jsonlogic)
[![Platform](https://img.shields.io/cocoapods/p/jsonlogic.svg?style=flat)](https://cocoapods.org/pods/jsonlogic)
[![codecov](https://codecov.io/gh/advantagefse/json-logic-swift/branch/master/graph/badge.svg)](https://codecov.io/gh/advantagefse/json-logic-swift)

A native Swift JsonLogic implementation. This parser accepts [JsonLogic](http://jsonlogic.com) 
rules and executes them. 

JsonLogic is a way to write rules that involve computations in JSON 
format, these can be applied on JSON data with consistent results. So you can share between server and clients rules in a common format. Original JS JsonLogic implementation is developed by Jeremy Wadhams.

## Instalation

#### Using CocoaPods

To use the pod in your project add in the Podfile:

    pod jsonlogic

To run the example project, just run:

    pod try jsonlogic    

#### Using Swift Package Manager

if you use Swift Package Manager add the following in dependencies:

        dependencies: [
        .package(
            url: "https://github.com/advantagefse/json-logic-swift", from: "1.0.0"
        )
    ]

## Usage

You simply import the module and either call the applyRule global method:

```swift
import jsonlogic

let rule =
"""
{ "var" : "name" }
"""
let data =
"""
{ "name" : "Jon" }
"""

//Example parsing
let result: String? = try? applyRule(rule, to: data)

print("result = \(String(describing: result))")
```

The ```applyRule``` will parse the rule then apply it to the ```data``` and try to convert the 
result to
 the 
inferred return 
type, 
if it fails an error will be thrown.

If you need to apply the same rule to multiple data then it will be better to parse the rule once.
You can do this by initializing a ```JsonRule``` object with the rule and then calling 
```applyRule```.

```swift

//Example parsing
let jsonlogic = try JsonLogic(rule)

var result: Bool = jsonlogic.applyRule(to: data1)
result = jsonlogic.applyRule(to: data2)
//etc..

```

## Examples

#### Simple
```Swift
let rule = """
{ "==" : [1, 1] }
"""

let result: Bool = try applyRule(rule)
//evaluates to true
```

This is a simple test, equivalent to `1 == 1`.  A few things about the format:

  1. The operator is always in the "key" position. There is only one key per JsonLogic rule.
  1. The values are typically an array.
  1. Each value can be a string, number, boolean, array (non-associative), or null

#### Compound
Here we're beginning to nest rules.

```Swift
let rule = """
  {"and" : [
    { ">" : [3,1] },
    { "<" : [1,3] }
  ] }
"""
let result: Bool = try applyRule(rule)
//evaluates to true
```

In an infix language this could be written as:

```Swift
( (3 > 1) && (1 < 3) )
```

#### Data-Driven

Obviously these rules aren't very interesting if they can only take static literal data. 
Typically `jsonLogic` will be called with a rule object and a data object. You can use the `var` 
operator to get attributes of the data object:

```Swift
let rule = """
  { "var" : ["a"] }
"""
let data = """
  { a : 1, b : 2 }
"""
let result: Int = try applyRule(rule, to: data)
//evaluates to 1
```

If you like, we support to skip the array around values:

```Swift
let rule = """
  { "var" : "a" }
"""
let data = """
  { a : 1, b : 2 }
"""
let result: Int = try applyRule(rule, to: data)
//evaluates to 1
```

You can also use the `var` operator to access an array by numeric index:

```js
jsonLogic.apply(
  {"var" : 1 },
  [ "apple", "banana", "carrot" ]
);
// "banana"
```

Here's a complex rule that mixes literals and data. The pie isn't ready to eat unless it's cooler than 110 degrees, *and* filled with apples.

```Swift
let rule = """
{ "and" : [
  {"<" : [ { "var" : "temp" }, 110 ]},
  {"==" : [ { "var" : "pie.filling" }, "apple" ] }
] }
"""
let data = """
  { "temp" : 100, "pie" : { "filling" : "apple" } }
"""

let result: Bool = try applyRule(rule, to: data)
//evaluates to true
```

### Custom operators

You can register a custom operator

```Swift
import jsonlogic
import JSON

// the key is the operator and the value is a closure that takes as argument
// a JSON and returns a JSON
let customRules =
    ["numberOfElementsInArray": { (json: JSON?) -> JSON in                                 
        switch json {
        case let .Array(array):
            return JSON(array.count)
        default:
            return JSON(0)
        }
    }]
    
let rule = """
    { "numberOfElementsInArray" : [1, 2, 3] }
"""
    
// The value is 3
let value: Int = try JsonLogic(rule, customOperators: customRules).applyRule()
```

### Other operators

For a complete list of the supported operators and their usages see [jsonlogic operators](http://jsonlogic.com/operations.html).

### Command Line Interface

Comming soon...

## Contributing

Making changes are welcome. 
If you find a bug please submit a unit test that reproduces it, before submitting the fix.

Because the project was created and build using the Swift PM there is no Xcode project file 
committed in the repo. If you need one you can generated by running ```genenate-xcodeproj.sh ``` 
in the terminal:

```
$ . generate-xcodeproj.sh
```

## Requirements


| iOS      | tvOS       | watchOS    | macOS      |
| :------: |:----------:|:----------:|:----------:|
| >=8.0    | >=10.0     | >=2.0      | >=10.12    |


## Author

Christos Koninis, c.koninis@afse.eu

## License

JsonLogic for Swift is available under the MIT license. See the LICENSE file for more info.
