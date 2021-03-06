![alt text](https://github.com/voltrue2/in-app-purchase/blob/develop/logo/75x75.png)

©Nobuyori Takahashi < <voltrue2@yahoo.com> >

[![Build Status](https://travis-ci.org/voltrue2/in-app-purchase.svg?branch=master)](https://travis-ci.org/voltrue2/in-app-purchase)

A node.js module for in-app purchase (in-app billing) for Apple, Google Play, Amazon Store, Roku, and Windows.

It supports Unity receipt also: [Unity Documentation](https://docs.unity3d.com/Manual/UnityIAPValidatingReceipts.html)

**NOTE** Unity receipt surpports the following: Apple, Google Play, and Amazon.

## Required node.js version

`0.12.0 >=`

## Online Demo and Documention

<a href="http://iap.gracenode.org" target="_blank">Online Demo</a>

## How to install

```
npm install in-app-purchase
```

## How to use

The module supports both Promise and callbacks.

```javascript
var iap = require('in-app-purchase');
iap.config({
	
	/* Configurations for Amazon Store */
	amazonAPIVersion: 2, // tells the module to use API version 2
	secret: 'abcdefghijklmnoporstuvwxyz', // this comes from Amazon
	// amazonValidationHost: http://localhost:8080/RVSSandbox, // Local sandbox URL for testing amazon sandbox receipts.
	
	/* Configurations for Apple */
	applePassword: 'abcdefg...', // this comes from iTunes Connect (You need this to valiate subscriptions)	
	
	/* Configurations for Google Play */
	googlePublicKeyPath: 'path/to/public/key/directory/' // this is the path to the directory containing iap-sanbox/iap-live files
	googleAccToken: 'abcdef...', // optional, for Google Play subscriptions
	googleRefToken: 'dddd...', // optional, for Google Play subscritions
	googleClientID: 'aaaa', // optional, for Google Play subscriptions
	googleClientSecret: 'bbbb', // optional, for Google Play subscriptions

	/* Configurations for Roku */
	rokuApiKey: 'aaaa...', // this comes from Roku Developer Dashboard
	
	/* Configurations all platforms */
	test: true, // For Apple and Googl Play to force Sandbox validation only
	verbose: true // Output debug logs to stdout stream 
});
iap.setup()
  .then(() => {
    // iap.validate(...) automatically detects what type of receipt you are trying to validate
    iap.validate(receipt).then(onSuccess).catch(onError);
  })
  .catch((error) => {
    // error...
  });

function onSuccess(validatedData) {
	// validatedData: the actual content of the validated receipt
	// validatedData also contains the original receipt
    var options = {
		ignoreCanceled: true, // Apple ONLY (for now...): purchaseData will NOT contain cancceled items
		ignoreExpired: true // purchaseData will NOT contain exipired subscription items
	};
	var purchaseData = iap.getPurchaseData(validateData, options);
}

function onError(error) {
	// failed to validate the receipt...
}
```

## Validate Receipts From Multiple Applications

You may feed different Google public key or Apple password etc to validate receipts of different applications with the same code:

### Windows is NOT Supported

### Google Public Key

```javascript
iap.config(configObject);
iap.setup()
  .then(() => {
    iap.validateOnce(receipt, pubKeyString).then(onSuccess).catch(onError);
  })
  .catch((error) => {
    // error...
  });
```

### Google Subscription

```javascript
iap.config(configObject);
iap.setup()
  .then(() => {
    var credentials = {
      clientId: 'xxxx',
      clientSecret: 'yyyy',
      refreshToken: 'zzzz'
    };
    iap.validateOnce(receipt, credentials).then(onSuccess).catch(onError);
  })
```

### Apple Subscription

```javascript
iap.config(configObject);
iap.setup()
  .then(() => {
    iap.validateOnce(receipt, appleSecretString).then(onSuccess).catch(onError);
  })
  .catch((error) => {
    // error...
  });
```

### Amazon

```javascript
iap.config(configObject);
iap.setup()
  .then(() => {
    iap.validateOnce(receipt, amazonSecretString).then(onSuccess).catch(onError);
  })
  .catch((error) => {
    // error...
  });
```

### Roku

```javascript
iap.config(configObject);
iap.setup()
  .then(() => {
    iap.validateOnce(receipt, rokuApiKeyString).then(onSuccess).catch(onError);
  })
  .catch((error) => {
    // error...
  });
```

## Google Play Public Key With An Environment Variable

You may not want to keep the public key files on your server(s).

The module also supports environment variables for this.

Instead of using `googlePublicKeyPath: 'path/to...'` in your configurations, you the following:

```
export=GOOGLE_IAB_PUBLICKEY_LIVE=PublicKeyHerePlz
export=GOOGLE_IAB_PUBLICKEY_SANDBOX=PublicKeyHerePlz
```

## Google In-app-Billing Set Up

To set up your server-side Android in-app-billing correctly, you must provide the public key string as a file from your Developer Console account.

**Reference:** <a href="https://developer.android.com/google/play/billing/billing_integrate.html#billing-security">Implementing In-app Billing</a>

Once you copy the public key string from the Developer Console account for your application, you simply need to copy and paste it to a file and name it `iap-live` as shown in the example above.

**NOTE:** The public key string you copy from the Developer Console account is actually a base64 string. You do NOT have to convert this to anything yourself. The module converts it to the public key automatically for you.

### Google Play Store API

To check expiration date or auto renewal status of an Android subscription, you should first setup the access to the Google Play Store API. You should follow these steps:

##### Part 1 - Get ClientID and ClientSecret
1. Go to https://play.google.com/apps/publish/
2. Click on `Settings`
3. Click on `API Access`
4. There should be a linked project already, if not, create one. If you have it, click it.
* You should now be at: https://console.developers.google.com/apis/library?project=xxxx
5. Under Mobile API's, make sure "Google Play Developer API is enabled".
6. Go back, on the left click on `Credentials`
7. Click `Create Credentials` button
8. Choose `OAuth Client ID`
9. Choose `Web Application`
 * Give it a name, skip the `Authorized JS origins`
 * Aadd this to `Authorized Redirect URIs`: https://developers.google.com/oauthplayground
 * Hit Save and copy the **clientID** and **clientSecret** somewhere safe.

##### Part 2 - Get Access and Refresh Tokens
1. Go to: https://developers.google.com/oauthplayground
2. On the right, hit the gear/settings.
3. Check the box: `Use your own OAuth credentials`
	* Enter in clientID and clientSecret
	* Close
4. On the left, find "Google Play Developer API v2"
 * Select "https://www.googleapis.com/auth/androidpublisher"
5. Hit Authorize Api's button
6. Save `Authorization Code` 
 * This is your: **googleAccToken**
7. Hit `Exchange Authorization code for token`
8. Grab: `Refresh Token`
 * This is your: **googleRefToken**

Now you are able to query for Android subscription status!

## Amazon App Store Reference

https://developer.amazon.com/docs/in-app-purchasing/iap-rvs-for-android-apps.html

## Windows Signed XML

in-app-purchase module supports the following algorithms:

### Canonicalization and Transformation Algorithms

- Exclusive Canonicalization http://www.w3.org/2001/10/xml-exc-c14n#

- Exclusive Canonicalization with comments http://www.w3.org/2001/10/xml-exc-c14n#WithComments

- Enveloped Signature transform http://www.w3.org/2000/09/xmldsig#enveloped-signature

### Hashing Algorithms

- SHA1 digests http://www.w3.org/2000/09/xmldsig#sha1

- SHA256 digests http://www.w3.org/2001/04/xmlenc#sha256

- SHA512 digests http://www.w3.org/2001/04/xmlenc#sha512

## HTTP Request Configurations

The module supports the same configurations as [npm request module] (https://www.npmjs.com/package/request#requestoptions-callback)

