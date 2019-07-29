 # Single Responsibility
> Class should have a single responsibility
- A single reason to change
- Group functionality 
- Separation of concerns
- Different classes handling different, independent tasks/problems

## Example

```swift
/// Journal only takes care of adding and removing journal entries
class Journal : CustomStringConvertible
{
    var entries = [String]()
    var count = 0

    func addEntry(_ text: String) -> Int
    {
        count += 1
        entries.append("\(count): \(text)")
        return count - 1
    }

    func removeEntry(_ index: Int)
    {
        entries.remove(at: index)
    }

    var description: String
    {
        return entries.joined(separator: "\n")
    }
}

/// Persistance is broken out into it's own class
class Persistence
{
    func saveToFile(_ journal: Journal, _ filename: String)
    {
        print("Saving \(journal) to \(filename)")
    }
}
```
