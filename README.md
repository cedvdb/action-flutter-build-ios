
Setups the macos environment to be able to build a flutter app targetting ios



# 1. Gather the certificate and provisioning profile

You'll need:
  
  - the p12 certificate
  - the password for the p12 certificate
  - the provisioning profile

[Here is how to get those](https://ioscodesigning.io/exporting-code-signing-files)


# 2. Add secrets values in your repository settings

Base64 encode the certificate and provisioning profiles

```
base64 -i BUILD_CERTIFICATE.p12 | pbcopy
```

and 

```
base64 -i PROVISIONING_PROFILE.mobileprovision | pbcopy
```

and add those values to your github repository secrets. The keychain password can be any random string.

![secrets](secrets.png)

 - IOS_BUILD_CERTIFICATE_BASE64 : This is the base64 version of your certificate.p12
 - IOS_BUILD_CERTIFICATE_PASSWORD : This is the password for certificate.p12
 - IOS_GITHUB_KEYCHAIN_PASSWORD : A random string of your choosing
 - IOS_MOBILE_PROVISIONING_PROFILE_BASE64 : The base64 version of your.mobileprovision file

# 3. Add export options

add the following `GithubActionsExportOptions.plist` file to the ios directory

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
	<dict>
		<key>generateAppStoreInformation</key>
		<false/>
		<key>manageAppVersionAndBuildNumber</key>
		<true/>
		<key>method</key>
		<string>app-store</string>
		<key>signingStyle</key>
		<string>manual</string>
		<key>provisioningProfiles</key>
		<dict>
			<key>{{ YOUR PACKAGE NAME }}</key>
			<string>{{ YOUR MOBILE PROVISION NAME }}</string>
		</dict>
		<key>stripSwiftSymbols</key>
		<true/>
		<key>teamID</key>
		<string>{{ YOUR TEAM ID }}</string>
		<key>testFlightInternalTestingOnly</key>
		<false/>
		<key>uploadSymbols</key>
		<true/>
	</dict>
</plist>
```

Replaces the values in the above files for:
  
  - TEAM_ID
  - PACKAGE_NAME
  - MOBILE PROVISION NAME

# 4. Usage


```yaml
name: Build and distribute

on:
  push:
    branches:
      - main

jobs:
  build:
    name: build
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2

      - uses: cedvdb/flutter-build-ios
        with:
          build-cmd: flutter build ipa --release --flavor dev --dart-define="flavor=dev" --export-options-plist=ios/GithubActionsExportOptions.plist
          certificate-base64: ${{ secrets.IOS_BUILD_CERTIFICATE_BASE64 }}
          certificate-password: ${{ secrets.IOS_BUILD_CERTIFICATE_PASSWORD }}
          provisioning-profile-base64: ${{ secrets.IOS_MOBILE_PROVISIONING_PROFILE_BASE64 }}
          keychain-password: ${{ secrets.IOS_GITHUB_KEYCHAIN_PASSWORD }}

          
      - name: Archive APK
        uses: actions/upload-artifact@v2
        with:
          name: release-apk
          # Try running the build locally with the build command to be sure of this path
          path: build/app/outputs/flutter-apk/app-dev-release.apk
```

Note: when using the build-cmd, use the `--export-options-plist=ios/GithubActionsExportOptions.plist` argument, so it uses the export option created in step #3.