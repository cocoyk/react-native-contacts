# React Native Contacts
Work in progress successor to react-native-addressbook. This is essentially a pre-alpha release. Expect breaking changes!

Rx support with [react-native-contacts-rx](https://github.com/JeanLebrument/react-native-contacts-rx)

#### Status
* Preliminary iOS and Android support
* API subject to revision, changelog in release notes  

| Feature | iOS | Android |
| ------- | --- | ------- |
| `getAll`  | ✔   | ✔ |
| `addContact` | ✔ | 😞 |
| `updateContact` | ✔ | 😞 |
| `deleteContact` | ✔ | 😞 |
| get with options | 😞 | 😞 |
| groups  | 😞 | 😞 |



## API
`getAll` (callback) - returns *all* contacts as an array of objects  
`addContact` (contact, callback) - adds a contact to the AddressBook.  
`updateContact` (contact, callback) - where contact is an object with a valid recordID  
`deleteContact` (contact, callback) - where contact is an object with a valid recordID  

## Usage
`getAll` is a database intensive process, and can take a long time to complete depending on the size of the contacts list. Because of this, it is recommended you access the `getAll` method before it is needed, and cache the results for future use.

Also there is a lot of room for performance enhancements in both iOS and android. PR's welcome!

```js
var Contacts = require('react-native-contacts')

Contacts.getAll((err, contacts) => {
  if(err && err.type === 'permissionDenied'){
    // x.x
  } else {
    console.log(contacts)
  }
})
```

## Example Contact Record
```js
{
  recordID: 1,
  familyName: "Jung",
  givenName: "Carl",
  middleName: "",
  emailAddresses: [{
    label: "work",
    email: "carl-jung@example.com",
  }],
  phoneNumbers: [{
    label: "mobile",
    number: "(555) 555-5555",
  }],
  thumbnailPath: "",
}
```
**NOTE**
* on Android the entire display name is passed in the `givenName` field. `middleName` and `familyName` will be `""`.

## Adding Contacts
Currently all fields from the contact record except for thumbnailPath are supported for writing
```js
var newPerson = {
  familyName: "Nietzsche",
  givenName: "Friedrich",
  emailAddresses: [{
    label: "work",
    email: "mrniet@example.com",
  }],
}

Contacts.addContact(newPerson, (err) => { /*...*/ })
```

## Updating and Deleting Contacts
```js
//contrived example
Contacts.getAll( (err, contacts) => {
  //update the first record
  let someRecord = contacts[0]
  someRecord.emailAddresses.push({
    label: "junk",
    email: "mrniet+junkmail@test.com",
  })
  Contacts.updateContact(someRecord, (err) => { /*...*/ })

  //delete the second record
  Contacts.deleteContact(contacts[1], (err) => { /*...*/ })
})
```
Update and delete reference contacts by their recordID (as returned by the OS in getContacts). Apple does not guarantee the recordID will not change, e.g. it may be reassigned during a phone migration. Consequently you should always grab a fresh contact list with `getContacts` before performing update and delete operations.

You can also delete a record using only it's recordID like follows: `Contacts.deleteContact({recordID: 1}, (err) => {})}`

## Getting started - iOS
1. `npm install react-native-contacts`
2. In XCode, in the project navigator, right click `Libraries` ➜ `Add Files to [your project's name]`
3. add `./node_modules/react-native-contacts/RCTContacts.xcodeproj`
4. In the XCode project navigator, select your project, select the `Build Phases` tab and in the `Link Binary With Libraries` section add **libRCTContacts.a**

## Getting started - Android
* In `android/settings.gradle`
```gradle
...
include ':react-native-contacts'
project(':react-native-contacts').projectDir = new File(settingsDir, '../node_modules/react-native-contacts/android')
```

* In `android/app/build.gradle`
```gradle
...
dependencies {
    ...
    compile project(':react-native-contacts')
}
```

* register module (in android/app/src/main/java/[your-app-namespace]/MainActivity.java)
```java
import com.rt2zz.reactnativecontacts.ReactNativeContacts; // <------ add import

public class MainActivity extends Activity implements DefaultHardwareBackBtnHandler {
  ......
    mReactInstanceManager = ReactInstanceManager.builder()
      .setApplication(getApplication())
      .setBundleAssetName("index.android.bundle")
      .setJSMainModuleName("index.android")
      .addPackage(new MainReactPackage())
      .addPackage(new ReactNativeContacts())              // <------ add package
      .setUseDeveloperSupport(BuildConfig.DEBUG)
      .setInitialLifecycleState(LifecycleState.RESUMED)
      .build();
  ......
}
```

* add Contacts permission (in android/app/src/main/AndroidManifest.xml)
(only add the permissions you need)
```xml
...
  <uses-permission android:name="android.permission.READ_CONTACTS" />
  <uses-permission android:name="android.permission.WRITE_CONTACTS" />
...
```

## Permissions Methods (iOS only, optional)
`checkPermission` (callback) - checks permission to use AddressBook.  
`requestPermission` (callback) - request permission to use AddressBook.  

Usage as follows:
```js
Contacts.checkPermission( (err, permission) => {
  // Contacts.PERMISSION_AUTHORIZED || Contacts.PERMISSION_UNDEFINED || Contacts.PERMISSION_DENIED
  if(permission === 'undefined'){
    Contacts.requestPermission( (err, permission) => {
      // ...
    })
  }
  if(permission === 'authorized'){
    // yay!
  }
  if(permission === 'denied'){
    // x.x
  }
})
```

These methods do **not** re-request permission if permission has already been granted or denied. This is a limitation in iOS, the best you can do is prompt the user with instructions for how to enable contacts from the phone settings page `Settings > [app name] > contacts`.

## Todo
- [ ] android feature parity
- [ ] migrate iOS from AddressBook to Contacts
- [ ] implement `get` with options
- [ ] groups support
