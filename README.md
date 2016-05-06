node-applesign
===============

AppleSigner is a NodeJS API for re-signing iOS applications.

Author
------

Sergi Alvarez aka pancake @ nowsecure.com

Dependencies
------------

* zip      - re-create IPA
* unzip    - decompress IPA
* codesign - sign and verify binary with new entitlements and identity

	codesign -v appPath

* security - get entitlements from mobileprovision

	security cms -D -i provisionPath

Future
------

* Use zip.js instead of system `zip` and `unzip` executables
* Reimplement the Apple code signing thing in pure Javascript
* Support xcarchives
* Use event model instead of callbacks

	Codesign.signIPA('ipafile').on('error', error_handle).on('ready', finished).start()

* Run that thing in the browser
* Profit

Usage
-----

	$ bin/ipa-resign.js
	Usage: codesign [--output new.ipa] [--identities] [--identity id]
		[--mobileprovision file] [--bundleid id]
		[input-ipafile]

List local codesign identities:

	bin/ipa-resign --identities

Resign an IPA with a specific identity:

	bin/ipa-resign --identity 1C4D1A442A623A91E6656F74D170A711CB1D257A foo.ipa

Change bundleid:

	bin/ipa-resign --bundleid org.nowsecure.testapp path/to/ipa

List mobile provisioning profiles:

	ls ~/Library/MobileDevice/Provisioning\ Profiles
	security cms -D -i embedded.mobileprovision   # Display its contents

Install mobileprovisioning in device:

	ideviceprovision install /path/to.mobileprovision

Define output IPA filename and install in device:

	bin/ipa-resign.js --output test.ipa
	ios-deploy -b test.ipa

Provisionings
-------------

In device:

	ideviceprovision list
	ideviceprovision install /path/to/provision

In System

	ls ~/Library/MobileDevice/Provisioning\ Profiles
	security find-identity -v -p codesigning

Show provisioning profile contents:

	security cms -D -i embedded.mobileprovision

API usage
---------

Here's a simple program that resigns an IPA:

```
const Applesign = require('node-applesign');
const as = new Applesign({
  file: '/path/to/app.ipa',
  identity: '81A24300FE2A8EAA99A9601FDA3EA811CD80526A',
  mobileprovision: '/path/to/dev.mobileprovision'
});
as.signIPA((err, data) => {
  if (err) {
    console.log(data);
  }
  console.log('ios-deploy -b /path/to/app-resigned.ipa');
});
```

To list the developer identities available in the system:

```
as.getIdentities((err, ids) => {
  if (err) {
    console.error(err, ids);
  } else {
    ids.forEach((id) => {
      console.log(id.hash, id.name);
    });
  }
});
```

Bear in mind that the Applesign object can tuned to use different
configuration options:

```
const options = {
  file: '/path/to/app.ipa',
  outfile: '/path/to/app-resigned.ipa',
  entitlement: '/path/to/entitlement',
  bundleid: 'app.company.bundleid',
  identity: 'hash id of the developer',
  mobileprovision: '/path/to/mobileprovision file'
};
```

Further reading
---------------

https://github.com/maciekish/iReSign
http://dev.mlsdigital.net/posts/how-to-resign-an-ios-app-from-external-developers/
