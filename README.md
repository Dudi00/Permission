<p align="center">
  <img src="https://github.com/delba/Permission/raw/assets/Permission@2x.png">
</p>

<p align="center">
  <a href="https://travis-ci.org/delba/Permission"><img alt="Travis Status" src="https://img.shields.io/travis/delba/Permission.svg"/></a>
  <a href="https://img.shields.io/cocoapods/v/Permission.svg"><img alt="CocoaPods compatible" src="https://img.shields.io/cocoapods/v/Permission.svg"/></a>
  <a href="https://github.com/Carthage/Carthage"><img alt="Carthage compatible" src="https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat"/></a>
</p>

**Permission** exposes a unified API to request permissions on iOS.

<p align="center">
    <a href="#usage">Usage</a> • <a href="#example">Example</a> • <a href="#installation">Installation</a> • <a href="#license">License</a>
</p>

## Usage

#### Permission

> [`Permission.swift`](https://github.com/delba/Permission/blob/master/Source/Permission.swift)
> [`PermissionStatus.swift`](https://github.com/delba/Permission/blob/master/Source/PermissionStatus.swift)

```swift
let permission: Permission = .Contacts

print(permission.status) // PermissionStatus.NotDetermined

permission.request { status in
    switch status {
    case .Authorized:    print("authorized")
    case .Denied:        print("denied")
    case .Disabled:      print("disabled")
    case .NotDetermined: print("not determined")
    }
}
```

**Supported permissions**:

> [`PermissionType.swift`](https://github.com/delba/Permission/blob/master/Source/PermissionType.swift)
> [`PermissionTypes/`](https://github.com/delba/Permission/tree/master/Source/PermissionTypes)

- [`Bluetooth`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/Bluetooth.swift)
- [`Camera`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/Camera.swift)
- [`Contacts`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/Contacts.swift)
- [`Events`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/Events.swift)
- [`Motion`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/Motion.swift)
- [`Microphone`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/Microphone.swift)
- [`Notifications`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/Notifications.swift)
- [`Photos`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/Photos.swift)
- [`Reminders`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/Reminders.swift)
- [`LocationAlways`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/LocationAlways.swift)
- [`LocationWhenInUse`](https://github.com/delba/Permission/blob/master/Source/PermissionTypes/LocationWhenInUse.swift)

#### PermissionAlert

> [`PermissionAlert.swift`](https://github.com/delba/Permission/blob/master/Source/PermissionAlert.swift)

When you first request a permission, a system alert is presented to the user.
If you request a permission that was denied/disabled, a `PermissionAlert` will be presented.
You might want to change the default `title`, `message`, `cancel` and `settings` text:

```swift
let alert = permission.deniedAlert // or permission.disabledAlert

alert.title    = "Please allow access to your contacts"
alert.message  = nil
alert.cancel   = "Cancel"
alert.settings = "Settings"
```

#### PermissionSet

> [`PermissionSet.swift`](https://github.com/delba/Permission/blob/master/Source/PermissionSet.swift)

Use a `PermissionSet` to check the status of a group of `Permission` and to react a permission is requested.

```swift
let permissionSet = PermissionSet(.Contacts, .Camera, .Microphone, .Photos)
permissionSet.delegate = self

print(permissionSet.status) // PermissionStatus.NotDetermined

// ...

func permissionSet(permissionSet: PermissionSet, willRequestPermission permission: Permission) {
    print("Will request \(permission)")
}

func permissionSet(permissionSet: PermissionSet, didRequestPermission permission: Permission) {
    switch permissionSet.status {
    case .Authorized:    print("all the permissions are granted")
    case .Denied:        print("at least one permission is denied")
    case .Disabled:      print("at least one permission is disabled")
    case .NotDetermined: print("at least one permission is not determined")
    }
}
```

#### PermissionButton

> [`PermissionButton`](https://github.com/delba/Permission/blob/master/Source/PermissionButton.swift)

A `PermissionButton` requests the permission when tapped and updates itself when its underlying permission status changes.

```swift
let button = PermissionButton(.Photos)
```

`PermissionButton` is a subclass of `UIButton`. All the getters and setters of `UIButton` have their equivalent in `PermissionButton`.

```swift
button.setTitles([
    .Authorized:    "Authorized",
    .Denied:        "Denied",
    .Disabled:      "Disabled",
    .NotDetermined: "Not determined"
])

// button.setAttributedTitles
// button.setTitleColors
// button.setTitleShadowColors
// button.setImages
// button.setBackgroundImages
// etc.
```

## Example

```swift
extension PermissionButton{
    /**
     For example purpouse
     
     - parameter title: title of resoursce you want to access
     */
    func setTitleAndColors(title:String){
        self.setTitles([
            .NotDetermined: "\(title) - NotDetermined",
            .Authorized:    "\(title) - Authorized",
            .Denied:        "\(title) - Denied"
            ])
        
        self.setTitleColors([
            .NotDetermined: UIColor(red:0.56, green:0.56, blue:0.58, alpha:1.00),
            .Authorized:    UIColor(red:0.30, green:0.85, blue:0.39, alpha:1.00),
            .Denied:        UIColor(red:1.00, green:0.58, blue:0.00, alpha:1.00)
            ])
    }
}

class PermissionsViewController: UIViewController,PermissionSetDelegate {
    
    @IBOutlet weak var statusLabel: UILabel!
    @IBOutlet weak var permissionButtonsView: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let contacts   = PermissionButton(.Contacts)
        let camera     = PermissionButton(.Camera)
        let microphone = PermissionButton(.Microphone)
        let photos     = PermissionButton(.Photos)
        
        contacts.setTitleAndColors("Contacts")
        camera.setTitleAndColors("Camera")
        microphone.setTitleAndColors("Microphone")
        photos.setTitleAndColors("Photos")
        
        let permissionSet = PermissionSet(contacts, camera, microphone, photos)
        
        permissionSet.delegate = self
        
        statusLabel.text = String(permissionSet.status)
        
        for subview in [contacts, camera, microphone, photos] {
            permissionButtonsView.addSubview(subview)
            arrangeButton(subview, inView: permissionButtonsView)
        }
        
    }
    
    //MARK: PermissionSetDelegate
    func permissionSet(permissionSet: PermissionSet, didRequestPermission permission: Permission) {
        statusLabel.text = String(permissionSet.status)
    }

    /**
     Fuction to arrange buttons in view for example
     
     - parameter button: button to arrage
     - parameter v:      view where will be putted
     */
    private func arrangeButton(button:UIButton, inView v:UIView){
        button.translatesAutoresizingMaskIntoConstraints = false
        var viewAbove = v
        var anchor = NSLayoutAttribute.Top
        if v.subviews.count > 1 {
            let almostLastView = v.subviews[v.subviews.count-2]
            viewAbove = almostLastView
            anchor = .Bottom
        }
        v.addConstraint(NSLayoutConstraint(
            item: button,
            attribute: .Width,
            relatedBy: .Equal,
            toItem: v,
            attribute: .Width,
            multiplier: 1,
            constant: 0))
        
        button.addConstraint(NSLayoutConstraint(item: button,
            attribute: .Height,
            relatedBy: .Equal,
            toItem: nil,
            attribute: .Height,
            multiplier: 1,
            constant: 30))
        
        v.addConstraint(NSLayoutConstraint(item: button,
            attribute: .Top,
            relatedBy: .Equal,
            toItem: viewAbove,
            attribute: anchor,
            multiplier: 1,
            constant: 8))
        
        v.addConstraint(NSLayoutConstraint(item: button,
            attribute: .CenterX,
            relatedBy: .Equal,
            toItem: v,
            attribute: .CenterX,
            multiplier: 1,
            constant: 0))
    }
}


```

<img align="center" src="https://raw.githubusercontent.com/delba/Permission/assets/permission.gif" />

## Installation

### Carthage

[Carthage](https://github.com/Carthage/Carthage) is a decentralized dependency manager that automates the process of adding frameworks to your Cocoa application.

You can install Carthage with [Homebrew](http://brew.sh/) using the following command:

```bash
$ brew update
$ brew install carthage
```

To integrate Permission into your Xcode project using Carthage, specify it in your `Cartfile`:

```ogdl
github "delba/Permission"
```

### Cocoapods

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects.

You can install it with the following command:

```bash
$ gem install cocoapods
```

To integrate Permission into your Xcode project using CocoaPods, specify it in your `Podfile`:

```ruby
use_frameworks!

pod 'Permission'
```

## License

Copyright (c) 2015 Damien (http://delba.io)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
