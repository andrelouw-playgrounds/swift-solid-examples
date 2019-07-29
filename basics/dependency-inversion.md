# Dependency Inversion Principle
> High level modules should not depend on low level modules but they both should depend on abstractions 
Abstractions should not depend on details but details should depend on abstractions 

Nothing to do with dependency injection

## Example
```swift
enum Relationship {
    case parent
    case child
}

class Person {
    var name: String
    init(_ name: String) {
        self.name = name
    }
}

let parent = Person("John")
let child1 = Person("Chries")
let child2 = Person("Matt")

// MARK: - Before
class RelationshipsBefore { // Low level module
    var relationships = [(Person, Relationship, Person)]()

    func add(parent: Person, child: Person) {
        relationships.append((parent, .parent, child))
        relationships.append((child, .child, parent))
    }
}

class ResearchBefore { // High level module
    // here we are creating a dependency on RelationshipsBefore (Low level)
    init(relationships: RelationshipsBefore) {
        let relations = relationships.relationships
        // Also relying on the internal working of the RelationshipsBefore class, if anything changes in the RelationshipsBefore class this implementation will need to change
        for r in relations where r.0.name == "John" && r.1 == .parent {
            print("John has a child called \(r.2.name)")
        }
    }
}

print("================== BEFORE ========================")
let relationshipsBefore = RelationshipsBefore()
relationshipsBefore.add(parent: parent, child: child1)
relationshipsBefore.add(parent: parent, child: child2)

let researchBefore = ResearchBefore(relationships: relationshipsBefore)


// MARK: - After
protocol RelationshipBrowser {
    func children(for name: String) -> [Person]
}

// creating dependeny on abstraction
class RelationshipsAfter: RelationshipBrowser { // Low level module
    var relationships = [(Person, Relationship, Person)]()

    func add(parent: Person, child: Person) {
        relationships.append((parent, .parent, child))
        relationships.append((child, .child, parent))
    }

    func children(for name: String) -> [Person] {
        return relationships
            // need to use all params otherwise swift complains so using $2 as well
            .filter { $0.name == name && $1 == .parent && $2 != nil }
            .map {$2}
    }
}

class ResearchAfter {
    // creating dependency on abstraction rather than low lever module
    init(relationships: RelationshipBrowser) {
        for p in relationships.children(for: "John") {
            print("John has a child called \(p.name)")
        }
    }
}

print("================== AFTER ========================")
let relationshipsAfter = RelationshipsAfter()
relationshipsAfter.add(parent: parent, child: child1)
relationshipsAfter.add(parent: parent, child: child2)

let researchAfter = ResearchAfter(relationships: relationshipsAfter)

```

