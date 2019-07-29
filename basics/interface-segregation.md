
  # Interface Segregation Principle
  >Clients should not be forced to depend upon interfaces that they do not use.

- Reduce the side effects and frequency of required changes by splitting the software into multiple, independent parts.
- Don’t put to  much into an interface, split into seperate interfaces 

```swift
struct Document {}

// MARK: - Before
protocol Machine {
    func print(d: Document) throws -> Void
    func scan(d: Document) throws -> Void
    func fax(d: Document) throws -> Void
}

enum NoRequiredFunctionality: Error {
    case doesNotPrint
    case doesNotScan
    case doesNotFax
}

/// Forcing Printer to implement scan and fax
class SimplePrinter: Machine {
    func print(d: Document) {
        // printing documents
    }

    func scan(d: Document) throws {
        throw NoRequiredFunctionality.doesNotScan
    }

    func fax(d: Document) throws {
        throw NoRequiredFunctionality.doesNotFax
    }
}

// MARK: - After
/// Spliting all the functionalites up
protocol Printer {
    func print(d: Document)
}

protocol Scanner {
    func scan(d: Document)
}

protocol Fax {
    func fax(d: Document)
}


class OrdinaryPrinter : Printer {
    func print(d: Document) {
        // printing away
    }
}

class Photocopier : Printer, Scanner {
    func print(d: Document) {
        // Printing
    }

    func scan(d: Document) {
        // scanning
    }
}

protocol MultiFunctionPrinter : Printer, Scanner, Fax {}

class MultiFunctionMachine : MultiFunctionPrinter {
    let printer: Printer
    let scanner: Scanner

    init(printer: Printer, scanner: Scanner) {
        self.printer = printer
        self.scanner = scanner
    }

    func print(d: Document) {
        printer.print(d: d) // Decorator pattern
    }

    func scan(d: Document) {
        scanner.scan(d: d) // Decorator pattern
    }

    func fax(d: Document) {
        // faxing
    }
}
```
