HealthKitManager is a (fairly) comprehensive collection of closure-based methods written in Swift 4, serving as a central `HealthKitManager` object that simplifies the data querying process some may find confusing.


## Adding to your project

Either clone this repo and copy the `HealthKitManager.swift` file into your project, or copy & paste the raw text to a new swift file.

## Usage

#### 1. Initialize the HealthKitManager
Inside your `UIViewController`:

```swift
import Foundation
import HealthKit

class ViewController: UIViewController {
  let hkm = HealthKitManager()
  ...
}
```
#### 2. Request Authorization
In `viewDidLoad`, use HealthKitManager to request authorization for the HealthKit [datatypes](https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier) you plan to be reading/writing.

```swift
override func viewDidLoad() {
  let readingTypes: Set<HKObjectType> = [.heartRateType,
                                        .restingHeartRateType,
                                        .workoutType,
                                        .stepsType,
                                        .variabilityType]

  // auth request
  hkm.requestAuthorization(
    readingTypes: readingTypes, writingTypes: nil) {
      // This is a closure that runs upon authorization "success".
      // Execute your class's methods that require HealthKit here.
      ...
  }
}
```

- "Success" is in quotations here because this closure will run regardless of whether the user actually authorized the data access or ignored the popup window.

#### 3. Querying Data
Right now, HealthKitManager has several closure-based methods for reading heart rate, resting heart rate, heart rate variability, steps, and workout data.  After authorizations is granted, these can be used anywhere in your app's environment.
```swift
public func heartRate(from: Date, to: Date,
  handler: @escaping ([HKQuantitySample]) -> ())
```

```swift
public func restingHeartRate(from: Date, to: Date,
  handler: @escaping ([HKQuantitySample]) -> ())
```

```swift
public func variability(from: Date, to: Date,
  handler: @escaping ([HKQuantitySample]) -> ())
```

```swift
// check examples for clarity, querying steps with HealthKit is weird
public func dailySteps(handler: @escaping (HKStatisticsCollection)
  -> ()) { ... }
```

```swift
public func workouts(from: Date, to: Date,
  handler: @escaping ([HKWorkout]) -> ())
```


#### 4. Example usage with the `HealthKitManager` object
_Example 1:_ Heart rate data from today
```swift
let calendar = Calendar.current
let startDate = calendar.startOfDay(for: Date())
hkm.heartRate(from: startDate, to: Date()) {
  (results) in  // results is an array of [HKQuantitySample]
  ...
  // example conversion to BPM:
  for result in results {
    let bpm = result.quantity.doubleValue(for: .heartRateUnit)
    ...
  }

}
```

_Example 2:_ Steps data from today

Steps are queried using an `HKStatisticsCollectionQuery`, so in your closure you must use an [`enumerateStatistics` iterator](https://developer.apple.com/documentation/healthkit/hkstatisticscollection/1615783-enumeratestatistics) to handle the step data.

```swift
let calendar = Calendar.current
let startDate = calendar.startOfDay(for: Date())
hkm.dailySteps { (results) in
  results.enumerateStatistics(from: startDate, to: Date()) {
    (statistics, stop) in
    // get total sum of steps
    guard let sum = statistics.sumQuantity() else { return }
    let steps = sum.doubleValue(for: .count())
    ...
}

```

## Credits
[Ben Zimring](https://github.com/benzimring)
