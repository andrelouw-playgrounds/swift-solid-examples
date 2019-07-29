# Dependency Inversion Example
In the `ModuleGoalWithSubGoalListTableViewCell` the `setUpCell` method was depending on a low level module where it should be depending on a abstraction.
### Before
`ModuleGoalWithSubGoalListTableViewCell.swift`
```swift
 public func setUpCell(with viewModel: ModuleGoalWithSubGoalListRowViewModel, actionButtonHandler: ActionButtonHandler?) {}
```

### After
A protocol (`ModuleGoalWithSubgoalListViewModelType`) was created to break the low level dependency:
`ModuleGoalWithSubGoalListTableViewCell.swift`
```swift
 public func setUpCell(with viewModel: ModuleGoalWithSubgoalListViewModelType, actionButtonHandler: ActionButtonHandler?) {}
```

And the old viewModel now depends on the abstraction as well:
```swift
class ModuleGoalWithSubGoalListRowViewModel: ModuleGoalWithSubgoalListViewModelType {}
```
