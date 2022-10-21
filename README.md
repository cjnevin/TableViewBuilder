# TableViewBuilder

Check out the below example to see how to use this package.

## Cells

```swift
class InsertCell: UITableViewCell {
  struct Values: ConfigurableTableViewRow, EditableTableViewRow {
    var editingStyle: UITableViewCell.EditingStyle { .insert }
    var cellType: UITableViewCell.Type = InsertCell.self
    var title: String

    func configure(cell: UITableViewCell, at indexPath: IndexPath, in tableView: UITableView) {
      (cell as? InsertCell)?.configure(with: self)
    }
  }

  func configure(with values: Values) {
    textLabel?.text = values.title
  }
}

class Cell: UITableViewCell {
  struct Values: ConfigurableTableViewRow, TappableTableViewRow, MovableTableViewRow {
    var cellType: UITableViewCell.Type = Cell.self
    var title: String
    var onTapHandler: () -> Void

    func configure(cell: UITableViewCell, at indexPath: IndexPath, in tableView: UITableView) {
      (cell as? Cell)?.configure(with: self)
    }

    func onMove(from indexPath: IndexPath, to newIndexPath: IndexPath) -> IndexPath {
      if indexPath.section != newIndexPath.section {
          return indexPath
      }
      return IndexPath(row: max(1, newIndexPath.row), section: newIndexPath.section)
    }

    func onTap(at indexPath: IndexPath, in tableView: UITableView) {
      onTapHandler()
      tableView.deselectRow(at: indexPath, animated: true)
    }
  }

  func configure(with values: Values) {
    textLabel?.text = values.title
  }
}
```

## Headers

```swift
class InfoHeaderView: UITableViewHeaderFooterView {
  struct Values: ConfigurableTableViewHeader {
    func configure(view: UITableViewHeaderFooterView, at section: Int, in tableView: UITableView) {
      (view as? InfoHeaderView)?.configure(with: self)
    }

    var viewType: UITableViewHeaderFooterView.Type = InfoHeaderView.self
    var title: String
  }

  private lazy var label = UILabel()

  override init(reuseIdentifier: String?) {
    super.init(reuseIdentifier: reuseIdentifier)
    addSubview(label) {
        $0.top(24).bottom(-8).horizontalEdges(16) == Superview()
    }
  }

  required init?(coder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }

  func configure(with values: Values) {
    label.textColor = .blue
    label.font = .boldSystemFont(ofSize: 15)
    label.text = values.title
  }
}
```

## Footers

```swift
class InfoFooterView: UITableViewHeaderFooterView {
  struct Values: ConfigurableTableViewFooter {
    func configure(view: UITableViewHeaderFooterView, at section: Int, in tableView: UITableView) {
      (view as? InfoFooterView)?.configure(with: self)
    }

    var viewType: UITableViewHeaderFooterView.Type = InfoFooterView.self
    var title: String
  }

  private lazy var label = UILabel()

  override init(reuseIdentifier: String?) {
    super.init(reuseIdentifier: reuseIdentifier)
    addSubview(label) {
        $0.edges(20) == Superview()
    }
  }

  required init?(coder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }

  func configure(with values: Values) {
    label.textColor = .red
    label.font = .italicSystemFont(ofSize: 15)
    label.text = values.title
  }
}
```

## ViewController

```swift
class ViewController: UIViewController {
  private lazy var tableView = TableView(style: .insetGrouped)
    .setOnSectionsUpdated { newSections in
        self.navigationItem.title = "Items \(newSections.flatMap { $0.rows }.count)"
    }
    .setOnInsertRow { [unowned self] in
        self.makeCell()
    }

  override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.
    layoutTableView()

    navigationItem.rightBarButtonItem = UIBarButtonItem(barButtonSystemItem: .edit, target: self, action: #selector(edit))
  }

  @objc private func edit(sender: UIBarButtonItem) {
    let inserted = tableView.currentSections.map { (section: TableViewSection) -> TableViewSection in
      var copy = section
      copy.rows.insert(InsertCell.Values(title: "Insert"), at: 0)
      return copy
    }
    tableView.sections {
      inserted
    }
    tableView.setIsEditing(true)
    navigationItem.rightBarButtonItem = UIBarButtonItem(barButtonSystemItem: .done, target: self, action: #selector(done))
  }

  @objc private func done(sender: UIBarButtonItem) {
    let filtered = tableView.currentSections.compactMap { (section: TableViewSection) -> TableViewSection? in
      var copy = section
      copy.rows = section.rows.filter { !($0 is InsertCell.Values) }
      if copy.rows.isEmpty {
          return nil
      }
      return copy
    }
    tableView.sections {
      filtered
    }

    tableView.setIsEditing(false)
    navigationItem.rightBarButtonItem = UIBarButtonItem(barButtonSystemItem: .edit, target: self, action: #selector(edit))
  }

  private var index: Int = 1
  func makeCell() -> TableViewRow {
    defer { index += 1 }
    let current = index
    return Cell.Values(title: "Cell \(index)") {
      print("Tapped \(current)")
    }
  }

  private func layoutTableView() {
    view.addSubview(tableView) {
      $0.edges == view.safeAreaLayoutGuide
      $0.sections {
        for section in 1...3 {
          TableViewSection {
            TableViewHeaderTitle("Header \(section)")
            for _ in 1...3 {
              makeCell()
            }
            TableViewFooterTitle("Footer \(section)")
          }
        }
      }
    }
  }
}
```
