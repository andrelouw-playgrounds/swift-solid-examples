# Open-closed / Single Responsibility / Interface segregation Example
The `ModuleGoalWithSubGoalListTableViewCell` was built with two purposes in mind, being a goal cell header of some sorts and then catering for a subgoal list.
you would typically pass in a response and it would figure out what type of subgoal list to show etc. 
It would use `GoalListCardViewController` to house the subgoal list and then `GoalListContentView` would be the individual list items. 
A few problems I encountered here was:
1. `ModuleGoalWithSubGoalListTableViewCell` was trying to do too much, determining a goal header and subgoal list, resulting in multiple responsibilities (breaking single responsiability)
2. There wasn’t an option to use the subgoal list on it’s own without having to go and create a new cell on its own, i.e. I was forced to deal with the goal header AND the subgoal list.  (Think this can be linked to breaking the Interface Segregation Principle)
3. When a new sub goal item type was added another check had to be added to the `GoalListContentView` to hide or show labels or buttons depending on the use case. This was starting to get messy. (Breaking the open closed principle as I had to go and modify the `GoalListContentView` class instead of extending it)

### Before:
`ModuleGoalWithSubGoalListTableViewCell.swift`
```swift
public class ModuleGoalWithSubGoalListTableViewCell: UITableViewCell {
    private var viewModel: ModuleGoalWithSubgoalListViewModelType?
    private var goalListViewController: GoalListCardViewController?

    @IBOutlet weak var containerView: UIView!
    @IBOutlet weak var listPlaceholderView: UIView!
    @IBOutlet weak var goalTitle: UILabel!
    @IBOutlet weak var goalSubtitle: UILabel!
    @IBOutlet weak var goalActionButton: MEMSecondaryButton!
    @IBOutlet weak var goalImageView: UIImageView!
    @IBOutlet weak var goalRewardLabel: UILabel!

    internal var actionHandler: ActionButtonHandler?

    public override func awakeFromNib() {
        super.awakeFromNib()
        configureStyle()
    }

    private func configureStyle() {
        selectionStyle = .none
        goalTitle.configure(with: MEMAppearanceHandler.shared().blackTextColor(),
                            font: MEMAppearanceHandler.shared().semiBoldFont(withSize: 17))
        goalSubtitle.configure(with: MEMAppearanceHandler.shared().charcoalGrayTextColor(),
                               font: MEMAppearanceHandler.shared().regularFont(withSize: 15))
        goalRewardLabel.configure(with: MEMAppearanceHandler.shared().whiteTextColor(),
                                  font: MEMAppearanceHandler.shared().boldFont(withSize: 20))
        goalActionButton.buttonColor = MEMAppearanceHandler.shared().discoveryVitalityTintColor()
    }

    public func setUpCell(with viewModel: ModuleGoalWithSubgoalListViewModelType, actionButtonHandler: ActionButtonHandler?) {
        self.viewModel = viewModel
        self.actionHandler = actionButtonHandler
        setUpImage() 
        setUpLabels()
        setUpSubGoals()
    }

    private func setUpImage() {
        if let image = UIImage(named: "coinGoldBackground", in: MembersActiveRewards.bundle, compatibleWith: nil) {
            goalImageView.image = image
        } else {
            goalImageView.isHidden = false
        }
    }

    private func setUpLabels() {
        goalTitle.text = viewModel?.title ?? ""
        goalRewardLabel.text = viewModel?.rewardAmount ?? ""

        if let subtitleText = viewModel?.subtitle {
            goalSubtitle.text = subtitleText
        } else {
            goalSubtitle.isHidden = true
        }

        if let actionTitleText = viewModel?.actionTitle {
            goalActionButton.setTitle(actionTitleText, for: .normal)
        } else {
            goalActionButton.isHidden = true
        }
    }

    fileprivate func setUpSubGoals() {
        if let subGoals = viewModel?.subGoals {
            containerView.isHidden = false
            addGoalList(for: subGoals)
        } else {
            containerView.isHidden = true
        }
    }

    private func addGoalList(for subGoals: [GoalListItem] ) {
        let viewController = GoalListCardViewController.viewController(with: subGoals)
        viewController.view.translatesAutoresizingMaskIntoConstraints = false
        listPlaceholderView.addSubview(viewController.view)
        viewController.view.constrainEdges(to: listPlaceholderView)
        goalListViewController = viewController
    }

    @IBAction func buttonTapped(_ sender: Any) {
        actionHandler?()
    }
}
```

`GoalListCardViewController.swift`
```swift
public class GoalListCardViewController: UIViewController, Nibloadable {
    @IBOutlet weak var containerStackView: UIStackView!
    private var listItems: [GoalListItem] = []

    public static func viewController(with items: [GoalListItem]) -> GoalListCardViewController {
        guard let viewController = GoalListCardViewController.loadFromNib() else { return GoalListCardViewController() }
        viewController.listItems = items
        return viewController
    }

    override public func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = MEMAppearanceHandler.shared().whiteSmokeColor()
        view.layer.cornerRadius = 10
        addItems()
    }

    public func addItems() {
        for (index, item) in listItems.enumerated() {
            containerStackView.addArrangedSubview(contentView(with: item))
            if index < listItems.count - 1 {
                containerStackView.addArrangedSubview(UIView.seperatorView())
            }
        }
    }

    private func contentView(with item: GoalListItem) -> UIView {
        guard let view = GoalListContentView.viewFromNib() else { return UIView() }
        view.setUp(item: item)
        return view
    }
}
```

`GoalListContentView.swift`
```swift
public struct GoalListItem {
    var title: String
    var value: String
    var detail: String?
    var style: GoalListCardStyles
    var icon: UIImage?
    var tintColor: UIColor?
    var actionButtonText: String?

    public init(title: String,
                value: String? = nil,
                detail: String? = nil,
                icon: UIImage? = nil,
                tintColor: UIColor? = nil,
                actionButtonText: String? = nil) {
        if value == nil && detail == nil && actionButtonText == nil {
            self.style = .simple
        } else if actionButtonText != nil {
            self.style = .actionable
        } else {
            self.style = .detailed
        }

        self.title = title
        self.value = value ?? "-"
        self.detail = detail
        self.icon = icon
        self.tintColor = tintColor
        self.actionButtonText = actionButtonText
    }
}

internal enum GoalListCardStyles {
    case detailed
    case simple
    case actionable
}

internal class GoalListContentView: UIView {
    @IBOutlet weak var titleLabelContainerView: UIView!
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var titleImageView: UIImageView!
    @IBOutlet weak var valueLabel: UILabel!
    @IBOutlet weak var detailLabel: UILabel!
    @IBOutlet weak var actionButton: UIButton!

    @IBOutlet weak var stackViewTopConstraint: NSLayoutConstraint!
    @IBOutlet weak var stackViewBottomConstraint: NSLayoutConstraint!
    
    let kHeightOfTopAndBottomPadding: CGFloat = 15.0
    let kHeightOfTopAndBottomPaddingForSimple: CGFloat = 10.0

    override internal func awakeFromNib() {
        super.awakeFromNib()
        backgroundColor = MEMAppearanceHandler.shared().clearColor()
        titleLabelContainerView.backgroundColor = MEMAppearanceHandler.shared().clearColor()
    }

    internal func setUp(item: GoalListItem) {
        titleLabel.text = item.title
        valueLabel.text = item.value

        if item.detail == nil {
            detailLabel.isHidden = true
        } else {
            detailLabel.text = item.detail
        }


        if let itemIcon = item.icon {
            titleImageView.image = itemIcon
            titleImageView.image = titleImageView.image?.withRenderingMode(.alwaysTemplate)
            titleImageView.tintColor = MEMAppearanceHandler.shared().successGreenColor()
        } else  {
            titleImageView.isHidden = true
        }

        if let itemButtonTitle = item.actionButtonText {
            actionButton.setTitle(itemButtonTitle, for: .normal)
            actionButton.tintColor = MEMAppearanceHandler.shared().discoveryVitalityTintColor()
        }

        switch item.style {
        case .detailed:
            styleForDetailed(tintColour: item.tintColor ?? MEMAppearanceHandler.shared().blackTextColor())
        case .simple:
            styleForSimple()
        case .actionable:
            styleForActionable()
        }
    }

    private func styleForDetailed(tintColour: UIColor) {
        actionButton.isHidden = true
        titleLabel.configure(with: MEMAppearanceHandler.shared().blackTextColor(),
                             font: MEMAppearanceHandler.shared().semiBoldFont(withSize: 17),
                             numberOfLines: 0)
        valueLabel.configure(with: tintColour,
                             font: MEMAppearanceHandler.shared().semiBoldFont(withSize: 20),
                             numberOfLines: 0)
        detailLabel.configure(with: MEMAppearanceHandler.shared().charcoalGrayTextColor(),
                              font: MEMAppearanceHandler.shared().regularFont(withSize: 15),
                              numberOfLines: 0)
        updateStackViewConstraints(with: kHeightOfTopAndBottomPadding)
    }

    private func styleForSimple() {
        actionButton.isHidden = true
        titleLabel.configure(with: MEMAppearanceHandler.shared().blackTextColor(),
                             font: MEMAppearanceHandler.shared().regularFont(withSize: 17),
                             numberOfLines: 0)
        valueLabel.isHidden = true
        detailLabel.isHidden = true
        updateStackViewConstraints(with: kHeightOfTopAndBottomPaddingForSimple)
    }

    private func styleForActionable() {
        actionButton.isHidden = false
        valueLabel.isHidden = true

        titleLabel.configure(with: MEMAppearanceHandler.shared().blackTextColor(),
                             font: MEMAppearanceHandler.shared().semiBoldFont(withSize: 17),
                             numberOfLines: 0)
        detailLabel.configure(with: MEMAppearanceHandler.shared().charcoalGrayTextColor(),
                              font: MEMAppearanceHandler.shared().regularFont(withSize: 15),
                              numberOfLines: 0)
        actionButton.titleLabel?.font = MEMAppearanceHandler.shared().regularFont(withSize: 17)
        updateStackViewConstraints(with: kHeightOfTopAndBottomPadding)
    }

    private func updateStackViewConstraints(with constant: CGFloat) {
        stackViewTopConstraint.constant = constant
        stackViewBottomConstraint.constant = constant
    }
}
```

### After
Because there wasn’t an option to use the subgoal list component on it’s own without having to take the goal header, and the use case required the subgoal list on its own, I broke the subgoal list item out in order to try and apply SOLID as far as possible:
Firstly, to try and stick to single responsibility, each subgoal item will have it’s own model and we will simplify the view to the bear minimum. The view will start off by hiding all labels and the each model will show the labels according to its type. This way we can reuse views that are more or less similar, but as soon a new label or button is introduced we wiil create a new view. In this approach the models and the views are also seperated so we can always swap out the views. 

The view then becomes:
`ModuleDetailedSubGoalItemView.swift`
```swift
public struct DetailedSubGoalItem {
    var title: String? { get }
    var value: String? { get }
    var message: String? { get }
    var tintColour: UIColor? { get }
}

class ModuleDetailedSubGoalItemView: UIView {
    private var item = DetailedSubGoalItem()

    // By default all outlets are set to hidden, have to be explicitly unhide them
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var valueLabel: UILabel!
    @IBOutlet weak var messageLabel: UILabel!

    override internal func awakeFromNib() {
        super.awakeFromNib()
        backgroundColor = MEMAppearanceHandler.shared().clearColor()
    }

    func setUp(with item: DetailedSubGoalItem) {
        self.item = item
        styleLabels()
        setUpLabel(titleLabel, with: item.title)
        setUpLabel(valueLabel, with: item.value)
        setUpLabel(messageLabel, with: item.message)
    }
}

private extension ModuleDetailedSubGoalItemView {
    func styleLabels() {
        titleLabel.configure(with: MEMAppearanceHandler.shared().blackTextColor(),
                             font: MEMAppearanceHandler.shared().semiBoldFont(withSize: 17),
                             numberOfLines: 0)
        valueLabel.configure(with: item.tintColour ?? MEMAppearanceHandler.shared().blackTextColor(),
                             font: MEMAppearanceHandler.shared().semiBoldFont(withSize: 20),
                             numberOfLines: 0)
        messageLabel.configure(with: MEMAppearanceHandler.shared().charcoalGrayTextColor(),
                              font: MEMAppearanceHandler.shared().regularFont(withSize: 15),
                              numberOfLines: 0)
    }

    func setUpLabel(_ label: UILabel, with text: String?) {
        guard let text = text else { return }
        label.isHidden = false
        label.text = text
    }
}
```

Each goal item type is then defined in an enum with an associated value that is ready to setup the view:
`SubGoalType.swift`
```swift
public enum SubGoalType {
    case detailed(_ item: DetailedSubGoal)
    case simple(_ item: SimpleSubGoal)
}

public struct DetailedSubGoal: DetailedSubGoalItem {
    var title: String? 
    var value: String?
    var message: String?
    var tintColour: UIColor?

    public init(title: String, value: String, message: String, tintColour: UIColor) {
        self.title = title
        self.value = value
        self.message = message
        self.tintColour = tintColour
    }
}

public struct SimpleSubGoal: SimpleSubGoalItem {
    var title: String?
    var icon: UIImage?

    public init(title: String? = nil, icon: UIImage? = nil) {
        self.title = title
		  self.icon = icon
    }
}
```

A factory will then match the type of cell with an appropriate view:
`ModuleSubGoalItemViewFactory.swift`
```swift
class ModuleSubGoalItemViewFactory {
    static func view(for subGoal: SubGoalType) -> UIView {
        switch subGoal {
        case .detailed(let item):
            return ModuleSubGoalItemViewFactory.detailedSubGoalView(with: item)
        }
    }
}

private extension ModuleSubGoalItemViewFactory {
    static func detailedSubGoalView(with item: DetailedSubGoal) -> UIView {
        guard let view = ModuleDetailedSubGoalItemView.viewFromNib() else { return UIView() }
        view.setUp(with: item)
        return view
    }
}
```

So this might seem like we duplicated the code from our the before section, but we have managed to split out the subgoal item, so it can now be used on it’s own or with a more complicated setup, this somewhat adheres to the `Interface Segregation Principle` by splitting the software into multiple, independent parts. although `Interface Segregation Principle` mostly refers to interfaces I think it helps by simplifying this solution. 

Secondly, if we need to add a new subgoal item that is very close to the detailed subogal item we don’t need to go and add a lot of if checks to hide certain labels, al we need to do is add the new type: 
`SubGoalType.swift`
```swift
public enum SubGoalType {
    case detailed(_ item: DetailedSubGoalItem)
    case simple(_ item: SimpleSubGoalItem)
	  case notSoDetailed(_ item: NotSoDetailedGoal)
}

// Here we create a struct that will only have valueso we care about, and when passing this to our resuable view the values we kept as nil will just stil hide. So effectively we created a new type, without modifying the view
public struct NotSoDetailedGoal: DetailedSubGoalItem {
    var title: String? 
    var value: String?
    var message: String?
    var tintColour: UIColor?

    public init(title: String, message: String, tintColour: UIColor) {
        self.title = title
        self.message = message
        self.tintColour = tintColour
    }
}
```

And lastly we add this type to the factory:
`ModuleSubGoalItemViewFactory.swift`
```swift
class ModuleSubGoalItemViewFactory {
    static func view(for subGoal: SubGoalType) -> UIView {
        switch subGoal {
        case .detailed(let item):
            return ModuleSubGoalItemViewFactory.detailedSubGoalView(with: item)
		  case .simple(let item):
            return ModuleSubGoalItemViewFactory.simpleSubGoalView(with: item)
        case .notSoDetailed(let item):
			  return ModuleSubGoalItemViewFactory.notSoDetailedSubGoalView(with: item)
        }
    }
}

private extension ModuleSubGoalItemViewFactory {
    static func notSoDetailedSubGoalView(with item: NotSoDetailedGoal) -> UIView {
        guard let view = ModuleDetailedSubGoalItemView.viewFromNib() else { return UIView() }
        view.setUp(with: item)
        return view
    }
}
```

In doing this we extend every time instead of modifying when need to add a new subgoal type that can reuse the detailed subgoal view. As a result adhering to the `Open-closed principle`

Lastly, if we add a new subgoal that is completely different from a detailed subgoal (lets imagine we are bringing buttons into the mix now that has to cater for action handlers etc…)  then we create a new view that caters for this type and map the subgoal type to the new view in the factory. 
that way we adhere to the single responsibility principle and avoid modifying our detailed subgoal view. 
