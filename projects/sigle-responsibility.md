# Single responsibility principle
In one of the projects I needed to do for my goals (Realm data binding) I started of with doing persistence inside of the view controller. Later on I broke out persistence into it’s own class separating persistence from the table view controller. 
### Before
All realm actions were sitting inside `CategoryTableViewController`
```swift
extension CategoryTableViewController {
    func loadCategories()  {
        categories = realm.objects(Category.self)
    }

    func save(_ category: Category) {
        try? realm.write {
            realm.add(category)
        }
    }

    func delete(_ category: Category) {
        try? realm.write {
            realm.delete(category)
        }
    }

    func addRealmObserver() {
        notificationToken = categories?.observe { (changes) in
            switch changes {
            case .initial:
                self.tableView.reloadData()
            case .update(_, _, _, _):
                self.tableView.reloadData()
            case .error(let error):
                print("Error: \(error)")
            }
        }
    }
}
```

### After
Created a Persistence class:

`PersistanceController.swift`
```swift
protocol PersistanceControllerDelegate {
    func didUpdateWithInitialChanges<Value>(_ controller: PersistanceController<Value>)
    func didUpdate<Value>(_ controller: PersistanceController<Value>, withDeletions deletions: [Int], withInsertions insertions: [Int], withModifications modifications: [Int])
}

extension PersistanceControllerDelegate {
    func didUpdateWithInitialChanges<Value>(_ controller: PersistanceController<Value>) {}
    func didUpdate<Value>(_ controller: PersistanceController<Value>, withDeletions deletions: [Int], withInsertions insertions: [Int], withModifications modifications: [Int]) {}
}

extension PersistanceControllerDelegate where Self: UITableViewController {
    func didUpdate<Value>(_ controller: PersistanceController<Value>) where Value : Object {
        tableView.reloadData()
    }

    func didUpdate<Value>(_ controller: PersistanceController<Value>, withDeletions deletions: [Int], withInsertions insertions: [Int], withModifications modifications: [Int]) where Value : Object {
        tableView.beginUpdates()
        tableView.insertRows(at: insertions.map { IndexPath(row: $0, section: 0) }, with: .automatic)
        tableView.deleteRows(at: deletions.map { IndexPath(row: $0, section: 0) }, with: .automatic)
        tableView.reloadRows(at: modifications.map { IndexPath(row: $0, section: 0) }, with: .automatic)
        tableView.endUpdates()
    }
}

class PersistanceController<Value: Object> {

    let realm = try! Realm()
    var delegate: PersistanceControllerDelegate?
    var notificationToken: NotificationToken?

    init() {}

    deinit {
        notificationToken?.invalidate()
    }

    func loadCategories() -> Results<Value> {
        return realm.objects(Value.self)
    }

    func save(_ value: Value) {
        try? realm.write {
            realm.add(value)
        }
    }

    func update(_ handler: () -> Void) {
        try? realm.write {
            handler()
        }
    }

    func updateWithoutNotifiying(_ handler: () -> Void) {
        realm.beginWrite()
        handler()
        if let notificationToken = notificationToken {
            try? realm.commitWrite(withoutNotifying: [notificationToken])
        } else {
            try? realm.commitWrite(withoutNotifying: [])
        }
    }

    func delete(_ value: Value) {
        try? realm.write {
            realm.delete(value)
        }
    }

    func addRealmObserver(for values: Results<Value>?) {
        notificationToken = values?.observe { [weak self] changes in
            guard let strongSelf = self else { return }
            switch changes {
            case .initial:
                strongSelf.delegate?.didUpdateWithInitialChanges(strongSelf)
            case .update(_, let deletions, let insertions, let modifications):
                strongSelf.delegate?.didUpdate(strongSelf, withDeletions: deletions, withInsertions: insertions, withModifications: modifications)
            case .error(let error):
                print("Error: \(error)")
            }
        }
    }
}
```

And the implementation inside the table view controller then becomes:

`CategoryTableViewController.swift`
```swift
class CategoryTableViewController: TodoTableViewController, PersistanceControllerDelegate {

    var persistance = PersistanceController<Category>() // Instantiate persistance class
    var categories: Results<Category>?	// still have object in controller for more contorl

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Realm Todo"
        persistance.delegate = self // set self as persistance delegate
        categories = persistance.loadCategories() // load objects via persitance class
        persistance.addRealmObserver(for: categories) // add observers via persistance class
    }

    @IBAction func didTapAddButton(_ sender: UIBarButtonItem) {
        showAddAlert { [weak self] title in
            let category = Category(with: title)
            self?.persistance.save(category) // save via persitance class
        }
    }

    override func deleteModel(at indexPath: IndexPath) {
        guard let category = categories?[indexPath.row] else { return }
        showDeleteAlert(for: category.title) { [weak self] in
            self?.persistance.delete(category) // delete via persitance class
        }
    }

    override func editModel(at indexPath: IndexPath) {
        guard let category = categories?[indexPath.row] else { return }
        showEditAlert(for: category.title) { [weak self] text in
            self?.persistance.update {	// update via persitance class
                category.title = text
            }
        }
    }
}
```
