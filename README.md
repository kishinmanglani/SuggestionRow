<img src="https://img.shields.io/badge/platform-iOS-blue.svg?style=flat" alt="Platform iOS" />
<a href="https://developer.apple.com/swift"><img src="https://img.shields.io/badge/swift3-compatible-4BC51D.svg?style=flat" alt="Swift 3 compatible" /></a>

## Contents
* [Introduction](#introduction)
* [Usage](#usage)
* [Customization](#customization)
* [Dependencies](#dependencies)
* [Requirements](#requirements)
* [Getting involved](#getting-involved)

## Introduction
`SuggestionRow` is a row extension for the [Eureka](https://github.com/xmartlabs/Eureka) iOS form builder that provides completion suggestions as the user is typing. There are two rows that can be instantiated:
- `SuggestionAccessoryRow<T>` - provides completion suggestions as tags in a horizontal `collectionView` displayed as the `inputAccessoryView` above the keyboard.
- `SuggestionTableRow<T>` - provides completion suggestions in a table below the cell.

Both classes are generic and can be used to provide completion suggestions for any type that conforms to the `SuggestionValue` protocol:

    public protocol SuggestionValue: Equatable, InputTypeInitiable {
        var suggestionString: String { get }
    }

`SuggestionRow` is based off of [`GooglePlacesRow`](https://github.com/EurekaCommunity/GooglePlacesRow)

## Usage
### Conform to `SuggestionValue` protocol
First, the type that you want to provide completion suggestions for must conform to the `SuggestionValue` protocol. For example, consider the following `Scientist struct`:

    struct Scientist {
        var id: Int
        var firstName: String
        var lastName: String

        init(id: Int, firstName: String, lastName: String) {
            self.id = id
            self.firstName = firstName
            self.lastName = lastName
        }
    } 

To conform to the `SuggestionValue` protocol, add a `suggestionString` method that specifies how `Scientist`s should be displayed as suggestions. This is the text that will either go in a tag in the `inputAccessoryView` or in a row in the suggestions `UITableView`. For `Scientist`s, displaying the full name is a good option, as shown below. `Scientist` must also conform to `Equatable` and `InputTypeInitiable` as required by `SuggestionValue`. `InputTypeInitiable` is a protocol defined by `Eureka` which requires an optional initializer that takes in a `String`. It's used to determine how to instatiate a value from the contents of a `UITextField`. In this case, the text that the user types in is used to filter existing values but not create new ones so the initializer can return nil.

    extension Scientist: SuggestionValue {
        // Required by `InputTypeInitiable`, can always return nil in the SuggestionValue context.
        init?(string stringValue: String) {
            return nil
        }
        
        // Text that is displayed as a completion suggestion.
        var suggestionString: String {
            return "\(firstName) \(lastName)"
        }
    }

To conform to the [`Equatable`](https://developer.apple.com/reference/swift/equatable) protocol, implement the `==` operator at a global scope:

    func == (lhs: Scientist, rhs: Scientist) -> Bool {
        return lhs.id == rhs.id
    } 

### Instantiate the row
In your form, add a row of type `SuggestionAccessoryRow<Scientist>` or `SuggestionTableRow<Scientist>`. You must define the row's `filterFunction` to determine where completion suggestions come from and how they are selected based on the user's input. The function you set as `filterFunction` must take in a `String` parameter (what the user has typed in) and return an array of `Scientist` (the valid suggestions). This function is called each time the user input field changes. The following example is also in `SuggestionExampleViewController`:

    let users: [Scientist] = [Scientist(id: 1, firstName: "Albert", lastName: "Einstein"),
                              Scientist(id: 2, firstName: "Isaac", lastName: "Newton")]
    ...                          
    form +++ Section("Table suggestions")
        <<< SuggestionTableRow<Scientist>() {
            $0.filterFunction = { text in
                users.filter({ $0.firstName.lowercaseString.containsString(text.lowercaseString) })
            }
            $0.placeholder = "Search for a famous scientist"
        }

## Customization

### `SuggestionAccessoryRow<T>`
`SuggestionAccessoryRow `uses a generic `SuggestionCollectionCell` cell whose generic parameter is the `UICollectionViewCell` class used in the `inputAccessoryView`.

* If you want to make minor visual changes to the suggestion cells, use the `customizeCollectionViewCell` callback.

* If you want to change the **layout of the collectionView** then you can use/modify/override the `collectionViewLayout` attribute in the `cellSetup` method when declaring the row.

* If you want to change something about the **collectionView** (e.g. its height, backgroundColor) then you can also do that in the `cellSetup` method. 

* If you want to **change the collection view cell of the inputAccessoryView** drastically, create your own row (`MySuggestionAccessoryRow`) with your own cell class which conforms to `EurekaSuggestionCollectionViewCell`.

### SuggestionTableRow<T>
`SuggestionTableRow` uses a generic `SuggestionTableCell` cell whose generic parameter is the `UITableViewCell` class used to create the cells displayed in a `UITableView` with the suggested options.

* If you want to make minor visual changes to the suggestion table row cells, use the `customizeTableViewCell` callback.

* You can customize attributes of the `tableView` that displays the options. You should do this in `cellSetup` and keep in mind that the frame of the tableView is reset each time the tableView is displayed.

* If you want to change these cells drastically, create your own row with your own cell class which conforms to `EurekaSuggestionTableViewCell`.

## Dependencies
* Eureka 

## Requirements
* iOS 8.0+
* Xcode 8.3+
* Swift 3

## Getting involved
* If you **want to contribute** please feel free to **submit pull requests**.
* If you **have a feature request** please **open an issue**.
* If you **found a bug** or **need help** please **check older issues before submitting an issue.**.
