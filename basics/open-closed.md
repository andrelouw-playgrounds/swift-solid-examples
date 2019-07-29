# Open-Closed Principle 
> Open for extension but closed for modification

If you go in to a class and modifying it instead of extending it you are breaking this principle

## Example
```swift
// MARK: - Product
enum Color
{
    case red
    case green
    case blue
}

enum Size
{
    case small
    case medium
    case large
    case yuge
}

class Product
{
    var name: String
    var color: Color
    var size: Size

    init(_ name: String, _ color: Color, _ size: Size)
    {
        self.name = name
        self.color = color
        self.size = size
    }
}

let apple = Product("Apple", .green, .small)
let money = Product("Money", .green, .small)
let tree = Product("Tree", .green, .large)
let house = Product("House", .blue, .large)
let products = [apple, tree, house]

// MARK: - Before
/// Constantly modifying the class to add functionality, breaking rule:
/// Open for extension closed for modification
class ProductFilter
{
    func filterByColor(_ products: [Product], _ color: Color) -> [Product]
    {
        var result = [Product]()
        for p in products
        {
            if p.color == color
            {
                result.append(p)
            }
        }
        return result
    }

    func filterBySize(_ products: [Product], _ size: Size) -> [Product]
    {
        var result = [Product]()
        for p in products
        {
            if p.size == size
            {
                result.append(p)
            }
        }
        return result
    }

    func filterBySizeAndColor(_ products: [Product],
                              _ size: Size, _ color: Color) -> [Product] {
        var result = [Product]()
        for p in products
        {
            if (p.size == size) && (p.color == color)
            {
                result.append(p)
            }
        }
        return result
    }
}

print("------------- No OCP ------------------")
let pf = ProductFilter()
print("Green products:")
for p in pf.filterByColor(products, .green) {
    print(" - \(p.name) is green")
}

print("Large products:")
for p in pf.filterBySize(products, .large) {
    print(" - \(p.name) is large")
}

print("Large and blue products:")
for p in pf.filterBySizeAndColor(products, .large, .blue) {
    print(" - \(p.name) is large and blue")
}


// MARK: - After
/// Protocols specifying behaviour
protocol Specification {
    associatedtype T
    func  isSatisfied(_ item: T) -> Bool
}

protocol Filter {
    associatedtype T
    func filter<Spec>(_ items: [T], by spec: Spec) -> [T] where Spec: Specification, Spec.T == T
}

/// Color spec
class ColorSpec: Specification {
    typealias T = Product
    let color: Color

    init(_ color: Color) {
        self.color = color
    }

    func isSatisfied(_ item: Product) -> Bool {
        return item.color == color
    }
}

/// Size spec
class SizeSpec: Specification {
    typealias T = Product
    let size: Size

    init(_ size: Size) {
        self.size = size
    }

    func isSatisfied(_ item: Product) -> Bool {
        return item.size == size
    }
}

class CombineSpecification<T, SpecA, SpecB> : Specification where
    SpecA: Specification,
    SpecB: Specification,
    SpecA.T == T,
    SpecB.T == T {

    let firstSpec: SpecA
    let secondSpec: SpecB

    init(first: SpecA, second: SpecB) {
        self.firstSpec = first
        self.secondSpec = second
    }

    func isSatisfied(_ item: T) -> Bool {
        return firstSpec.isSatisfied(item) && secondSpec.isSatisfied(item)
    }
}

/// This however does not work 🤷‍♂️
class ListSpecification<T, Spec> : Specification where Spec: Specification, Spec.T == T {
    let specs: [Spec]

    init(specs: [Spec]) {
        self.specs = specs
    }

    func isSatisfied(_ item: T) -> Bool {
        for spec in specs {
            if !spec.isSatisfied(item) {
                return false
            }
        }
        return true
    }
}

class NewProductFilter: Filter {
    typealias T = Product

    func filter<Spec>(_ items: [Product], by spec: Spec) -> [Product] where Spec : Specification, Spec.T == T {
        var result = [Product]()
        for item in items {
            if spec.isSatisfied(item) {
                result.append(item)
            }
        }
        return result
    }
}

print("------------- With OCP ------------------")
let npf = NewProductFilter()
print("Green products:")
for p in npf.filter(products, by: ColorSpec(.green)) {
    print(" - \(p.name) is green")
}

print("Large products:")
for p in npf.filter(products, by: SizeSpec(.large)) {
    print(" - \(p.name) is large")
}

print("Large and blue products:")
for p in npf.filter(products, by: CombineSpecification(first: ColorSpec(.blue), second: SizeSpec(.large))) {
    print(" - \(p.name) is large and blue")
}

```
