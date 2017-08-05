# Auto Update
A simple Protocol Oriented Framework for writing easily-updatable, data-driven views.

## Protocols
I’m using a Protocol called `Unit` because a game I’m making involves war stuff.

### Model Protocol
	protocol Unit {
	    var name: String { get set }
	    var id: String { get }
	    var status: UnitStatus { get set }
	}

We will also need to be able to equate them.
	func ==(lhs: Unit?, rhs: Unit?) -> Bool {
	    return (lhs?.id == rhs?.id)
	}

Let’s create a simple `struct` to conform to our `Unit` protocol.
	struct Hero: Unit {
	    var id: String
	    var name: String
	    var status: UnitStatus
	}

### View Protocol
Next is our view protocol for displayed a `Unit`. This will be shockingly called `UnitView`.
	protocol UnitView {
	    var unit: Unit? { get }
	    func prepare(withUnit unit: Unit?)
	}
You can add whatever you’d like into this protocol, but the magic happens in the `prepare(withUnit unit: Unit?)`. Put all of your view updating code in `prepare(withUnit unit: Unit?)` and will be good to go.

We need to extend the `UnitView` protocol for UIViews (you can also do UIViewControllers as well).
	extension UnitView where Self: UIView {
	    func autoUpdate() {
	        NotificationCenter.default.addObserver(forName: NSNotification.Name.objectUpdated, object: nil, queue: OperationQueue.main) {
	            [weak self] (notification: Notification) in
	            
	            guard let unit = notification.userInfo?["unit"] as? Unit else {
	                return
	            }
	            
	            if self?.unit == unit {
	                self?.unit = unit
	                self?.prepare(withUnit: unit)
	            }
	        }
	    }
	}
Once the `autoUpdate()` function is called in your view initialization, your views (or View Controllers) will update themselves.

Here is an example of a `UICollectionViewCell` conforming to `UnitView` ready to be auto updated.
	class UnitCollectionViewCell: UICollectionViewCell, UnitView {
	    
	    var unit: Unit?
	    
	    @IBOutlet weak var nameLabel: UILabel!
	    @IBOutlet weak var statusLabel: UILabel!
	    
	    override func awakeFromNib() {
	        super.awakeFromNib()
	        autoUpdate()
	    }
	    
	    func prepare(withUnit unit: Unit?) {
	        self.nameLabel.text = unit?.name
	        self.statusLabel.text = unit?.face()
	    }
	    
	    override func prepareForReuse() {
	        super.prepareForReuse()
	        prepare(withUnit: unit)
	    }
	}

## Data Store
Another key component to the Auto Update framework is a data store to send out notifications to our `UnitView`s.

Here is all you need for a simple data store.
	class DataStore {
	    
	    static let shared = DataStore()
	    
	    func update(object: Unit?) {
	        guard let updatedUnit = object else {
	            return
	        }
	        
		// save the object in whatever way you need to
	        
	        // notifies our UnitViews that a Unit was updated
	        NotificationCenter.default.post(name: NSNotification.Name.objectUpdated, object: nil, userInfo: [Constants.objectKey: updatedUnit])
	    }
	}

## Diagram
![](https://www.dropbox.com/s/1m49xoo7kc9qusv/Diagram.png?raw=1)
