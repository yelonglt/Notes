#Swift基本语法

###Optional
```
enum Optional<T> {
    case None
    case Some(T)
}

let x: String? = nil
//... is the same as ...
let y = Optional<String>.None

let zx: String? = "Hello"
//... is the same as ...
let zy = Optional<String>.Some("Hello")

```

###Array
```
var a = Array<String>()
//... is the same as ...
var a = [String]()

let animals = ["Giraffe", "Cow", "Doggie", "Bird"]
animals.append("Ostrich") //won't complie,animals is immutable(because is let)
let animal = animals[5] //crash(array index out of bounds)

for animal in animals {
    print("\(animal) \n")
}

```

###Dictionary
```
var teams = Dictionary<String, Int>()
//... is the same as ...
var a = [String:Int]()

teams = ["Stanford":1,"Cal":2]

for (key, value) in teams {
    print("\(key) == \(value)")
}

```
###Range
```
struct Range<T> {
    var startIndex: T
    var endIndex: T
}

let array = ["a", "b", "c", "d", "e"]
let subarray1 = array[2...3]
let subarray2 = array[2..<3]

```

###NSObject

###NSNumber
```
let value = NSNumber.init(value: 35.5)
let intversion = value.intValue
let boolverson = value.boolValue //value==0 false  其他 ture

```
###NSDate

###NSData




