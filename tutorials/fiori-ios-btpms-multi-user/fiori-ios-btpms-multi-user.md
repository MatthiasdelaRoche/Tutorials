---
title: Enable Multi-User Mode for iOS Application
description: Learn how to configure the assistant generated application to enable the multi-user mode (one device, multiple users with secure access).
auto_validation: true
time: 60
tags: [ tutorial>intermediate, topic>mobile, products>sap-mobile-services, operating-system>ios, products>sap-business-technology-platform]
primary_tag: products>sap-btp-sdk-for-ios
---

## Prerequisites
- You have [Set Up a BTP Account for Tutorials](group.btp-setup). Follow the instructions to get an account, and then to set up entitlements and service instances for the following BTP services.
    - **SAP Mobile Services**
- You have [created your first application with SAP BTP SDK for iOS](group.ios-sdk-setup).

## Details
### You will learn
- How to create a native iOS application that supports multiple users on the same device
- How to modify the SAP BTP SDK for iOS assistant generated application


---

[ACCORDION-BEGIN [Step 1: ](The real world use case)]
An air carrier organisation uses an iOS application built using SAP BTP SDK for iOS to keep a track of an aircraft's vital information (Fuel Level, Tyre Pressure, etc.) before each flight. The application must support offline use-case to comply with the network regulations at the airport. Since the airline has flights departing round the clock, it deploys ground staff in three 8-hour shifts. To maximise efficiency, the organisation wants ground staff to share mobile devices.

The ground staff members want a solution that is reliable even in the absence of network. They also aren't keen on logging out and logging in every time a shift ends, as they believe this could lead to erroneous data.

In this tutorial, you will learn how to enhance your [SAP BTP SDK for iOS Assistant](https://developers.sap.com/trials-downloads.html?search=SAP+BTP+SDK+for+iOS) generated application to create an offline enabled application that supports multiple users.

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 2: ](Enable multi-user mode in mobile services cockpit)]

1. In your mobile services account, click Mobile Applications &rarr; Native/Hybrid &rarr; **<Your Mobile Application configuration>**.

    ![Mobile Service Cockpit App Config](img_2_1.png)

2. Under Assigned Features, click **Mobile Settings Exchange**.

    ![Mobile Settings Exchange](img_2_2.png)

    > Assigned features can be found under the Info tab.

3. Under Shared Devices section, enable the *Allow Upload of Pending Changes from Previous User (Enable Multiple User Mode):* checkbox.

    ![Mobile Settings Exchange](img_2_3.png)

    > You must enable this checkbox if you are building an offline capable multi-user application.

[VALIDATE_1]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 3: ](Configure trust)]

In the given scenario any pending changes done by a user should be uploaded before another user signs in. Thus, you must configure trust to enable upload of pending changes from previous users of mobile applications.

1. Open your mobile services account.

    ![Mobile Service Cockpit Home](img_3_1.png)

2. In the Side Navigation Bar, click Settings &rarr; **Security**.

    ![Mobile Service Security](img_3_2.png)

3. Click **Metadata Download**.

    ![Mobile Service Metadata Download Button](img_3_3.png)

    > `NoAuth` authentication type is not supported. Even if multi-user mode is turned on, an application using the `NoAuth` authentication type will revert to single user mode.

4. Go to your sub account on SAP BTP.

    ![BTP Cockpit Home](img_3_4.png)

5. In the Side Navigation Bar, click Security &rarr; **Trust Configuration**.

    ![BTP Trust Configurations](img_3_5.png)

6. Click **New Trust Configuration**.

    ![New Trust Configurations Button](img_3_6.png)

7. Click **Upload**, and select the XML file downloaded in the earlier step.

    ![XML Upload Button](img_3_7.png)

8. Enter a name, and click **Save**.

    ![Rename & Save Step](img_3_8.png)

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 4: ](Configure app parameters)]

1. In Xcode, Open **`AppParameters.plist`**.

    > Ensure that you have completed the prerequisites before starting this step.

2. Add a new parameter by providing the following key/value.

    |  Key   | Type | Value |
    |  :------------- | :------------- | :------------- |
    |  User Mode | String | Multiple |

      ![AppParameters.plist Xcode View](img_4_2.png)  

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 5: ](Modify AppDelegate for multi-user onboarding flow and get multi-user callbacks)]

1. Open AppDelegate.swift

2. Update `initializeOnboarding` to configure `OnboardingSessionManager` to include `MultiUserOnboardingIDManager`. It will trigger multi-user flow.

```swift

func initializeOnboarding() {
    let presentationDelegate = ApplicationUIManager(window: self.window!)
    self.onboardingErrorHandler = OnboardingErrorHandler()
    self.sessionManager = OnboardingSessionManager(presentationDelegate: presentationDelegate, flowProvider: self.flowProvider, onboardingIDManager: MultiUserOnboardingIDManager(), delegate: self.onboardingErrorHandler)
    UserManager.register(self)
    presentationDelegate.showSplashScreenForOnboarding { _ in }

    self.onboardUser()
}
```
3. Add a private variable to track the user change status.

```swift
/// Boolean to ensure addUser/switchUser happened
private var userDidChange = false

```
4. Update `applicationWillEnterForeground` function

```swift
func applicationWillEnterForeground(_: UIApplication) {
    // Triggers to show the passcode screen
    OnboardingSessionManager.shared.unlock() { error in
        guard let error = error else {
            if self.userDidChange {
                self.afterOnboard()
                self.userDidChange = false
            }
            return
        }

        self.onboardingErrorHandler?.handleUnlockingError(error)
    }
}
```
5. Add the following code for multi-user callbacks.

```swift
extension AppDelegate: UserEventObserving {

    func userAdded(with onboardingID: UUID) {
        logger.debug("Called: userAdded")
        self.userDidChange = true
    }

    func userSwitched(to onboardingID: UUID) {
        logger.debug("Called: userSwitched")
        self.userDidChange = true
    }

}
```

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 6: ](Modify OnboardingFlowProvider and ODataOnboardingStep for reset passcode scenario)]


1. Open `OnboardingFlowProvider.swift` file.

2. Add `resetPasscode` case for the **flow** function.
```swift
public func flow(for _: OnboardingControlling, flowType: OnboardingFlow.FlowType, completionHandler: @escaping (OnboardingFlow?, Error?) -> Void) {
    switch flowType {
    case .onboard:
        completionHandler(self.onboardingFlow(), nil)
    case let .restore(onboardingID):
        completionHandler(self.restoringFlow(for: onboardingID), nil)
    case let .resetPasscode(onboardingID):
        completionHandler(self.resetPasscodeFlow(for: onboardingID), nil)
    case let .reset(onboardingID):
        completionHandler(self.resettingFlow(for: onboardingID), nil)
    case .background(onboardingID: let onboardingID):
        completionHandler(self.backgroundFlow(for: onboardingID), nil)
    @unknown default:
        break
    }
}
```

4. Add the following property and method.

```swift
public var resetPasscodeSteps: [OnboardingStep] {
    return self.onboardingSteps
}

func resetPasscodeFlow(for onboardingID: UUID) -> OnboardingFlow {
    let steps = self.resetPasscodeSteps
    let context = OnboardingContext(presentationDelegate: OnboardingFlowProvider.modalUIViewControllerPresenter)
    let flow = OnboardingFlow(flowType: .resetPasscode(onboardingID: onboardingID), context: context, steps: steps)
    return flow
}
```
5. Open ODataOnboardingStep.swift file.

6. Add `resetPasscode` function for `resetPasscode` scenario.

```swift
public func resetPasscode(context: OnboardingContext, completionHandler: @escaping (OnboardingResult) -> Void) {
    self.configureOData(using: context, completionHandler: completionHandler)
}
```

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 7: ](Configure OData for offline scenarios)]

## Step7: Configure OData for offline scenarios

1. Open `ODataControlling.swift` file

2. Import `SAPOfflineOdata`

```swift
import SAPOfflineOData
```

3. Add a new protocol for `configureOData` function:

```swift
func configureOData(sapURLSession: SAPURLSession, serviceRoot: URL, onboardingID: UUID, offlineParameters: OfflineODataParameters) throws
```
4. Open ODataOnboardingStep.Swift.

5. Import `SAPOfflineOdata`

```swift
import SAPOfflineOData
```

6. Add offline store name's key value.

```swift
    let offlineStoreNameKey: String = "SAP.OfflineOData.MultiUser"
```

7. Add a new function `offlineStoreID` that generates a UUID if the offline store name is nil. Otherwise returns the existing UUID.

```swift
private func offlineStoreID() -> UUID {
    var offlineStoreName: String? = UserDefaults.standard.value(forKey: self.offlineStoreNameKey) as? String
    if offlineStoreName == nil {
     offlineStoreName = UUID().uuidString
     UserDefaults.standard.set(offlineStoreName, forKey: self.offlineStoreNameKey)
    }
    let offlineStoreNameID: UUID = UUID(uuidString: offlineStoreName!)!
    return offlineStoreNameID
}
```
8. Update the reset function to pass the `offlineStoreID()` instead of `context.onboardingID`

```swift
public func reset(context: OnboardingContext, completionHandler: @escaping () -> Void) {
    defer { completionHandler() }
    do {
        try ESPMContainerOfflineODataController.removeStore(for: offlineStoreID())
    } catch {
        self.logger.error("Remove Offline Store failed", error: error)
    }
}
```

9. Add a new function `getOfflineODataParameters` to determine the new parameters for the user who is logging in.

```swift
private func getOfflineODataParameters(using context: OnboardingContext, completionnHandler: @escaping (OfflineODataParameters) -> Void) {
    var currentUser: String? = nil
    var forceUploadOnUserSwitch: Bool = false
    var storeEncryptionKey: String? = nil

    if let onboardedUser = UserManager().get(forKey: context.onboardingID), let userId = onboardedUser.infoString {
      currentUser = userId
      storeEncryptionKey = try? context.credentialStore.get(String.self, for: EncryptionConfigLoader.encryptionKeyID)
      if let enabled = (context.info[.sapcpmsSharedDeviceSettings] as? SAPcpmsSharedDeviceSettings)?.allowUploadPendingChangesFromPreviousUser {
          forceUploadOnUserSwitch = enabled
      }
      let offlineParameters = OfflineODataParameters()
      offlineParameters.currentUser = currentUser
      offlineParameters.forceUploadOnUserSwitch = forceUploadOnUserSwitch
      offlineParameters.storeEncryptionKey = storeEncryptionKey
      completionnHandler(offlineParameters)
    } else {
      fatalError("Failed to fetch user information!")
    }
}
```

10. Update the `configureOData` function to determine the user mode, and configure parameters accordingly.


```swift
private func configureOData(using context: OnboardingContext, completionHandler: @escaping (OnboardingResult) -> Void) {
  let semaphore: DispatchSemaphore =  DispatchSemaphore(value: 0)

    var offlineParameters: OfflineODataParameters = OfflineODataParameters()
    if UserManager.userMode == .multiUser {
      self.getOfflineODataParameters(using: context) { parameters in
          offlineParameters = parameters
          semaphore.signal()
      }
    } else {
        semaphore.signal()
    }
    semaphore.wait()
    let banner = topBanner()
    let group = DispatchGroup()
    var odataControllers = [String: ODataControlling]()
    let destinations = FileConfigurationProvider("AppParameters").provideConfiguration().configuration["Destinations"] as! NSDictionary

    let eSPMContainerOfflineODataDelegateSample = OfflineODataDelegateSample(for: "ESPMContainer", with: banner)
    odataControllers[ODataContainerType.eSPMContainer.description] = ESPMContainerOfflineODataController(delegate: eSPMContainerOfflineODataDelegateSample)

    for (odataServiceName, odataController) in odataControllers {
      group.enter()
      let destinationId = destinations[odataServiceName] as! String
      // Adjust this path so it can be called after authentication and returns a HTTP 200 code. This is used to validate the authentication was successful.
      let configurationURL = URL(string: (context.info[.sapcpmsSettingsParameters] as! SAPcpmsSettingsParameters).backendURL.appendingPathComponent(destinationId).absoluteString)!

      do {
          try odataController.configureOData(sapURLSession: context.sapURLSession, serviceRoot: configurationURL, onboardingID: offlineStoreID(), offlineParameters: offlineParameters)
          let connectivityStatus = ConnectivityUtils.isConnected()
          self.logger.info("Network connectivity status: \(connectivityStatus)")
          odataController.openOfflineStore(synchronize: connectivityStatus) { error in
              if let error = error {
                  completionHandler(.failed(error))
                  return
              }
              self.controllers[odataServiceName] = odataController
              group.leave()
          }
      } catch {
          completionHandler(.failed(error))
      }
    }
    group.notify(queue: .main) {
      completionHandler(.success(context))
    }
}
```

13. Open `ESPMContainerOfflineODataController.Swift`

12. Update `configureOData` function to accept `OfflineODataParameters`.

```swift
public func configureOData(sapURLSession: SAPURLSession, serviceRoot: URL, onboardingID: UUID, offlineParameters: OfflineODataParameters = OfflineODataParameters()) throws {
    offlineParameters.enableRepeatableRequests = true

    // Configure the path of the Offline Store
    let offlinePath = try ESPMContainerOfflineODataController.offlineStorePath(for: onboardingID)
    try FileManager.default.createDirectory(at: offlinePath, withIntermediateDirectories: true)
    offlineParameters.storePath = offlinePath

    // Setup an instance of delegate. See sample code below for definition of OfflineODataDelegateSample class.
    let offlineODataProvider = try! OfflineODataProvider(serviceRoot: serviceRoot, parameters: offlineParameters, sapURLSession: sapURLSession, delegate: delegate)
    try configureDefiningQueries(on: offlineODataProvider)
    self.dataService = ESPMContainer(provider: offlineODataProvider)
}
```


[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 8: ](Handle offline OData sync failure)]


1. Open `ESPMContainerOfflineODataController.Swift`

2. Update the error cases to include `syncFailed`.

```swift
public enum Error: Swift.Error {
  case cannotCreateOfflinePath
  case storeClosed
  case syncFailed
}
```

3. Update `openOfflineStore` function to catch the sync error.

```swift
  public func openOfflineStore(synchronize: Bool, completionHandler: @escaping (Swift.Error?) -> Void) {
    if !self.isOfflineStoreOpened {
        // The OfflineODataProvider needs to be opened before performing any operations
        self.dataService.open { error in
            if let error = error {
                self.logger.error("Could not open offline store.", error: error)
                if (error.code == -10425 || error.code == -10426) {
                    completionHandler(Error.syncFailed)
                } else {
                    completionHandler(error)
                }
                return
            }
            self.isOfflineStoreOpened = true
            self.logger.info("Offline store opened.")
            if synchronize {
                // You might want to consider doing the synchronization based on an explicit user interaction instead of automatically synchronizing during startup
                self.synchronize(completionHandler: completionHandler)
            } else {
                completionHandler(nil)
            }
        }
    } else if synchronize {
        // You might want to consider doing the synchronization based on an explicit user interaction instead of automatically synchronizing during startup
        self.synchronize(completionHandler: completionHandler)
    } else {
        completionHandler(nil)
    }
}
```

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 9: ](Multi-user error handling)]

## Step9: Multi-user error handling

1. Open OnboardingErrorHandler.swift

2. Update `onboardingController` function to handle application specific error handling.

```swift
public func onboardingController(_ controller: OnboardingControlling, didFail flow: OnboardingFlow, with error: Error, completionHandler: @escaping (OnboardingErrorDisposition) -> Void) {
    switch flow.flowType {
    case .onboard, .resetPasscode:
        self.onboardFailed(with: error, completionHandler: completionHandler)
    case .restore:
        self.restoreFailed(with: error, controller: controller, context: flow.context, completionHandler: completionHandler)
    default:
        completionHandler(.retry)
    }
}
```

3. Update `onboardFailed` function to handle duplicate user in case of add user and user mismatch in case of reset passcode.

```swift
private func onboardFailed(with error: Error, completionHandler: @escaping (OnboardingErrorDisposition) -> Void) {
    switch error {
    case WelcomeScreenError.demoModeRequested:
        completionHandler(.stop(error))
        return
    default:
        showAlertWith(error: error)
    }

    func showAlertWith(error: Error) {
        let alertController = UIAlertController(
            title: LocalizedStrings.Onboarding.failedToLogonTitle,
            message: error.localizedDescription,
            preferredStyle: .alert
        )
        switch error {
        case UserManagerError.userAlreadyExists(with: let onboardingID):
            alertController.addAction(UIAlertAction(title: "Restore", style: .default) { _ in
                self.switchToDuplicateUserWith(onboardingID: onboardingID, completionHandler: completionHandler)
            })
        case UserManagerError.userMismatch(with: let id):
            if let idNotNil = id {
                alertController.addAction(UIAlertAction(title: "Restore", style: .default) { _ in
                    self.switchToDuplicateUserWith(onboardingID: idNotNil, completionHandler: completionHandler)
                })

            } else {
                alertController.addAction(UIAlertAction(title: "Onboard", style: .default) { _ in
                    self.switchToDuplicateUserWith(onboardingID: nil, completionHandler: completionHandler)
                })

            }
        default:
            alertController.addAction(UIAlertAction(title: LocalizedStrings.Onboarding.retryTitle, style: .default) { _ in
                completionHandler(.retry)
            })
        }

        DispatchQueue.main.async {
            guard let topViewController = ModalUIViewControllerPresenter.topPresentedViewController() else {
                fatalError("Invalid UI state")
            }
            topViewController.present(alertController, animated: true)
        }
    }
}
```

4. Add a new function `switchToDuplicateUserWith` to set `transientUser` accordingly.

```swift
private func switchToDuplicateUserWith(onboardingID: String?, completionHandler: @escaping (OnboardingErrorDisposition) -> Void) {
    if let id = onboardingID {
        let user = UserManager().get(forKey: UUID(uuidString: id)!)
        user?.onboardingStatus = .restoreInProgress
        UserManager.transientUser = user
    } else {
        UserManager.transientUser = nil
    }
    completionHandler(.retry)
}
```

5. Update `restoreFailed` function to handle multiple-user errors during restore.

```swift
private func restoreFailed(with error: Error, controller: OnboardingControlling, context: OnboardingContext, completionHandler: @escaping (OnboardingErrorDisposition) -> Void) {
    let alertController = UIAlertController(title: nil, message: nil, preferredStyle: .alert)

    switch error {
    case StoreManagerError.cancelPasscodeEntry, StoreManagerError.skipPasscodeSetup, StoreManagerError.resetPasscode:
        resetOnboarding(context.onboardingID, controller: controller, completionHandler: completionHandler)
        return
    case StoreManagerError.passcodeRetryLimitReached:
        alertController.title = LocalizedStrings.Onboarding.passcodeRetryLimitReachedTitle
        alertController.message = LocalizedStrings.Onboarding.passcodeRetryLimitReachedMessage
    case ApplicationVersioningError.inactive:
        alertController.title = LocalizedStrings.Onboarding.failedToLogonTitle
        alertController.message = error.localizedDescription
        alertController.addAction(UIAlertAction(title: LocalizedStrings.Onboarding.retryTitle, style: .default) { _ in
            completionHandler(.retry)
        })
    case NorthwindEntitiesOfflineODataController.Error.syncFailed:
        let feedbackScreen = FUIFeedbackScreen.createInstanceFromStoryboard()
        feedbackScreen.messageNameLabel.text = "Sample Error Screen"
        feedbackScreen.messageDetailLabel.text = "Sample Label"
        feedbackScreen.messageActionButton.setTitle("Sample Button Name", for: UIControlState.normal)
        feedbackScreen.navigationController?.navigationBar.topItem?.title = "Error"
        feedbackScreen.didTapActionButton = {
            context.presentationDelegate.dismiss { error in
                if let error = error {
                    print(error.localizedDescription)
                }
            }
        }

        context.presentationDelegate.present(feedbackScreen) { error in
            if let error = error {
                print(error.localizedDescription)
            }

        }
        return
    default:
        alertController.title = LocalizedStrings.Onboarding.failedToLogonTitle
        alertController.message = error.localizedDescription
        alertController.addAction(UIAlertAction(title: LocalizedStrings.Onboarding.retryTitle, style: .default) { _ in
            completionHandler(.retry)
        })
    }

    alertController.addAction(UIAlertAction(title: LocalizedStrings.Onboarding.resetTitle, style: .destructive) { _ in
        self.resetOnboarding(context.onboardingID, controller: controller, completionHandler: completionHandler)
    })

    DispatchQueue.main.async {
        guard let topViewController = ModalUIViewControllerPresenter.topPresentedViewController() else {
            fatalError("Invalid UI state")
        }
        topViewController.present(alertController, animated: true)
    }
}
```




[DONE]
[ACCORDION-END]


[ACCORDION-BEGIN [Step 10: ](Build and run the application)]

1. In the menu bar, click Product &rarr; **Build**.

    ![Xcode Build](img_11_1.png)

    > Any errors you see will be cleared after building the project.

2. Select a suitable simulator/device and run your project.

    ![Xcode Run](img_11_2.png)


[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 11: ](Onboard multiple users)]

1. Click **Start**.

    ![iOS App Start](img_12_1.png)

2. Select **Default Identity Provider**.

    ![iOS App Identity Provider Selection](img_12_2.png)

3. Enter *username* and *password*, and click **Log On**.

    ![iOS App Log in](img_12_3.png)

4. Click **Allow Data**.

    ![iOS App Allow Data](img_12_4.png)

5. Click **Allow Usage**.

    ![iOS App Allow Usage](img_12_5.png)

6. Choose a *passcode*, and click **Next**.

    ![iOS App Choose Passcode](img_12_6.png)

    > Biometric authentication is not supported. The biometric screen will not be shown in the onboarding or unlock processes.

    > No passcode policy is not supported. A default passcode policy will be used if the server has disabled it.

7. Enter the *passcode* again, and click **Done**.

    ![iOS App Confirm Passcode](img_12_7.png)

8. Terminate the app or Send the app to background.

    ![iOS App Background](img_12_8.png)

9. In the sign in screen, click **Switch or Add User**.

    ![iOS App Sign in screen](img_12_9.png)

10. Click ![Add Users Icon](img_12_10_ico.png) to add a new user.
    > Ensure all users being on-boarded have access to the BTP account. You can configure this in the users section under security on your SAP BTP sub-account.
    > ![Create User ](img_12_10_note.png)

11. Follow the onboarding flow for the second user.

    ![iOS App Second user Sign in flow](img_12_11.gif)

12. Terminate the app or Send the app to background.

    ![iOS App Background](img_12_12.png)

13. In the sign in screen, click **Switch or Add User**.

    ![iOS App Sign in screen](img_12_13.png)

14. Select a different user.

    ![iOS App User select](img_12_14.png)

15. Enter the selected user's passcode.

    ![iOS App User passcode](img_12_15.png)

[VALIDATE_4]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 12: ](Try offline scenarios)]

1. Sign into User A's account.

    ![iOS App User passcode](img_13_1.png)

2. Turn off your network.

    > If you are running a simulator, turn off your parent system's WiFi.

3. Update an entry.

    ![iOS App field update](img_13_3.gif)

4. Turn on your network.

5. Sign in using User B's account.

    ![iOS App field update](img_13_5.png)

6. Verify the change done by user A.

    ![iOS App verify update](img_13_6.png)

In this step you've seen how the changes done by User A are not lost even when they were done in the absence of network.

Congratulations on completing the tutorial!

[DONE]
[ACCORDION-END]


---
