Sure, here's an example code snippet in React Native that demonstrates how to use the React Native SMS Retriever API to automatically retrieve an OTP (One-Time Password) sent via SMS, without requiring any SMS read permissions. Note that this approach may not work on some newer versions of Android, since Google has made some changes to the SMS Retriever API. 

1. Install the `react-native-sms-retriever` package using either yarn or npm in your project directory:

```
yarn add react-native-sms-retriever
```

or

```
npm install --save react-native-sms-retriever
```

2. In your `AndroidManifest.xml` file, add the following lines to specify the broadcast receiver for the SMS Retriever API:

```xml
<application>

  //...

  <receiver
    android:name="com.reactnativesmsretriever.SmsBroadcastReceiver"
    android:exported="true">
    <intent-filter>
      <action android:name="com.google.android.gms.auth.api.phone.SMS_RETRIEVED"/>
    </intent-filter>
  </receiver>

  //...

</application>
```

3. In your React Native component, use the `requestPhoneNumber()` method of the `react-native-sms-retriever` package to initiate the SMS retrieval process. This method requests the phone number of the user and returns a promise that resolves with the hash string that can be used to retrieve SMS messages:

```javascript
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  Button,
  NativeModules,
  PermissionsAndroid,
  Platform,
} from 'react-native';

const SMSRetriever = NativeModules.SmsRetriever;

const App = () => {
  const [phoneNumber, setPhoneNumber] = useState('');
  const [hashString, setHashString] = useState('');
  const [otp, setOtp] = useState('');

  const requestPhoneNumber = async () => {
    try {
      if (Platform.OS === 'android') {
        const granted = await PermissionsAndroid.request(
          PermissionsAndroid.PERMISSIONS.READ_PHONE_STATE,
          {
            title: 'Phone State Permission',
            message: 'This app needs access to your phone state in order to retrieve SMS messages.',
          },
        );
        if (granted === PermissionsAndroid.RESULTS.GRANTED) {
          const result = await SMSRetriever.requestPhoneNumber();
          setPhoneNumber(result.number);
          setHashString(result.hash);
        } else {
          console.log('Phone state permission denied.');
        }
      } else {
        console.log('Platform not supported.');
      }
    } catch (e) {
      console.log(e);
    }
  };

  const handleSmsRetrieved = (event) => {
    console.log(event);
    setOtp(event.message);
  };

  const startSmsRetriever = async () => {
    try {
      const registered = await SMSRetriever.startSmsRetriever(hashString);
      console.log(`SMS retrieval ${registered ? 'started' : 'failed'}.`);
    } catch (e) {
      console.log(e);
    }
  };

  return (
    <View>
      <Text>Enter your phone number:</Text>
      <TextInput
        value={phoneNumber}
        onChangeText={(text) => setPhoneNumber(text)}
      />
      <Button title="Request Phone Number" onPress={requestPhoneNumber} />
      <Button title="Start OTP Retrieval" onPress={startSmsRetriever} />
      <Text>Your OTP: {otp}</Text>
    </View>
  );
};

export default App;
```

Here's what the code does:

1. The `PermissionsAndroid` package is imported to request the `READ_PHONE_STATE` permission required to access the user's phone number.

2. The `react-native-sms-retriever` package is imported and its `requestPhoneNumber()` method is called to request the user's phone number and the corresponding hash string.

3. When the user clicks the "Request Phone Number" button, the `requestPhoneNumber()` method is called, prompting the user for the necessary permissions. If permissions are granted, the user's phone number and hash string are updated in the component's state.

4. When the user clicks the "Start OTP Retrieval" button, the `startSmsRetriever()` method is called to start the OTP retrieval process. The hash string is used to start the `SmsRetrieverClient` for retrieving the SMS message.

5. If a SMS containing a 6-digit OTP is received, the message is extracted and displayed in the `otp` state variable using the `handleSmsRetrieved()` function.

React Native provides a simple and efficient way for developers to code applications, however, it must be considered that the functionality of this project is deprecated, meaning that the compatibility is subject to change using React Native or updating Android's Google Play Services Sdk may disrupt its operation.
