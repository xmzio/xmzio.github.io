---
layout: post
title: Business cards with Passbook and iBeacons
excerpt: How to create Passbook-based business cards that can be shared with a barcode or a URL, and that use iBeacons to alert people when they're nearby.
assetsurl: assets/posts/2014-passbook-business-cards

---

The IBM Digital Experience Conference in Anaheim opens today, and [Base22 will have an exhibit space and five sessions](http://base22.com/wps/portal/home/about-us/news/all-news/digital-experience-2014) loaded with information on enterprise-web technical and business topics.

![Xavier's business card]({{ site.url }}/{{ page.assetsurl }}/screenshot-xaviers-pass.png){: .float-left} Since many Base22'ers have iPhones, and given the recent IBM + Apple announcements, I thought it would be fun to create some iBeacon-enabled Passbook business cards for my fellow coworkers who will be attending the conference.

The [Passbook Programming Guide](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/PassKit_PG/Chapters/Introduction.html) is the official document to go. It has all the information needed for building Passbook passes.

First I had to download the programming guide [companion file](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/PassKit_PG/Passbook%20Companion%20Files.zip). It's a zip file containing several Passbook sample passes and the _Signpass_ app for OS X. The Signpass app has to be built (an Xcode project is provided), and the executable will be used to sign the pass bundles.

There are 5 types of passes, each one with a different fixed layout:

* Boarding pass
* Coupon
* Event ticket
* Store card
* Generic pass

![Personal business card sketch]({{ site.url }}/{{ page.assetsurl }}/sketch-personal.png){: .float-right} The Generic pass format seemed adequate for the personal business cards:

It has a company logo in the top left corner, a square picture in the middle called the _thumbnail_, a main field that can be used for the person's name, and a couple of extra fields where I included the person's title, email address and Twitter handle. There's also space for a barcode where I encoded the pass URL.

![Company business card sketch]({{ site.url }}/{{ page.assetsurl }}/sketch-company.png){: .float-right} In addition to the personal business cards, I also made one for the company using the Store card format. I picked the Store card format because it allows you to set a prominent image from side to side, called the _strip_.

All passes can have fields with additional information on the back, and these fields can be presented as hyperlinks. This is nice, because you can add an email address and when the user taps it, the compose window of the Mail application will pop-open with the recipient address pre-populated, just as expected.

Phone numbers can be hyperlinks too. When touched, a popup asks the user for confirmation to dial that phone number, then the Phone application can start the call.


### Before the fun begins: Creating the certificate

A pass must be signed before we can start "debugging" it, so it's a good idea to get the certificate-related tasks out of the way first.

1. Login to Member Center, then go to _Certificates, Identifiers and Profiles_ / _Identifiers_ / _Pass Type IDs._
2. Create a new _Pass Type ID._
   * The description can be something indicating it's a business card and specifying the person's name, like "Xaviers business card" (note that the description field won't accept apostrophes.)
   * The ID should be a reverse-domain identifier, starting with the keyword _pass._ For example: `pass.com.base22.businesscard.xavier`.
3. Edit the newly created Pass Type ID, click _Create Certificate_ and follow the instructions. You will be creating a CSR with the Keychain Access app, uploading it, generating the certificate and downloading the certificate to your Mac (double-click the downloaded certificate to add it to your Mac's Keychain.)

Also, while at the Member Center, take note of your _Team ID_. This will be needed when creating the pass file. The Team ID is an alphanumeric, 10-character code, and can be found in the _Your Account_ page, under _Developer Account Summary._


### Preparing the pass bundle

The pass bundle is a folder with the _.pass_ extension. It contains the image files for the icon, logo, and thumbnail or strip, plus the _pass.json_ file, where all the fields and characteristics are specified (think of it as the pass source code.)

The easiest way to create a new pass is by duplicating one of the existing sample passes and replacing the image files with the real ones.

I had to create the following images for my passes:

|---------------+------------------------------------------------------------------------------+-----------------+-----------------|
| Name          | Description                                                                  | Width           | Height          |
|---------------|------------------------------------------------------------------------------|-----------------|-----------------|
| icon.png      | Used for location or time based notifications in the lock screen.            | 29 points       | 29 points       |
| logo.png      | Displayed at the top left corner of the pass.                                | 160 points max. | 50 points max.  |
| thumbnail.png | Used in the Generic pass; displayed at the right side of the main field.     | 90 points max.  | 90 points max.  |
| strip.png     | Used in the Store card pass; displayed under the logo, extends side to side. | 320 points      | 123 points max. |
|---------------+------------------------------------------------------------------------------+-----------------+-----------------|

These images must be created for retina and non-retina displays. That means that there's two versions of each file: One with the specified name, and another with the _@2x_ suffix (e.g. `logo@2x.png`) that's twice in pixel dimensions (If logo.png is 160 x 50 pixels, logo@2x.png must be 320 x 100 pixels.)


### Writing the pass.json file

Modifying the existing pass.json file from the sample pass is straightforward. All field names are self-explanatory.

First set the Team ID, the Pass Type ID and the serial number fields. These form the composed unique identifier. Only the serial number value will change when there's a new version of the pass.

For the personal business cards, I used the primary field for the person's name, the title would display in the secondary field, and the Twitter ID and email address would go in the auxiliary fields.

![Stack of business cards]({{ site.url }}/{{ page.assetsurl }}/screenshot-stack.png){: .float-right} Later I added a header field to display the first name. Since the plan was to have multiple business cards stacked in Passbook, I thought a short identifier in the top right corner of each card would be helpful.

For the company business card I didn't use the primary field, because I didn't want any text on top of the strip image.


### The pass download URL and the barcode

I wanted that people could easily share their cards by showing the barcodes, either by placing them in the final slide of their slideshows, or inviting visitors at the exhibit space to scan it with their iPhones.

The barcode block in the pass.json file looks like this:

{% highlight JavaScript %}
"barcode": {
	"message": "https://wiki.base22.com/cards/xavier.pkpass",
	"format": "PKBarcodeFormatPDF417",
	"messageEncoding": "iso-8859-1",
	"altText": "Scan with Passbook"
},
{% endhighlight %}

There's two requisites for the webserver that will be serving the final pass files:

1. It has to be an HTTPS webserver, with a certificate of a trusted CA (a self-signed certificate will not work.)
2. It must send the correct Content-type HTTP header when serving the pass files. The Passbook MIME type: `application/vnd.apple.pkpass` should be associated with the file extension _.pkpass_.


### The magic of iBeacons

The nicest thing about Passbook passes, is that either a notification or the entire pass can be displayed on the iPhone lock screen depending on certain factors.

For instance, I have used boarding passes from airlines that automatically show in the lock screen a few minutes before boarding time. This is really handy because I only have to get my iPhone from my pocket and present it to the airline personnel. No need to search for the boarding pass through my email.

![Notification in lock screen]({{ site.url }}/{{ page.assetsurl }}/screenshot-lock-screen.png){: .float-right} For the business cards, I wanted an alert on the lock screen whenever that person is nearby, because networking is a big deal in these conferences.

For this I associated each one of my Estimote iBeacons with a different pass.

Each iBeacon broadcasts 3 numbers: A Proximity UUID, a major identifier and a minor identifier. Typically all iBeacons from the same organization share the same Proximity UUID. Major identifiers are used to distribute beacons across separate areas, for example, different stores of the same chain, while minor identifiers are used to identify all the beacons in the same store.

We need to specify the 3 numbers of the beacon in the pass.json file. It will look like this:

{% highlight JavaScript %}
"beacons": [{
	"major": 12345,
	"minor": 67890,
	"proximityUUID": "A0000000-F000-4000-A000-2000000F0000",
	"relevantText": "Xavier is here... Say hi!"
}]
{% endhighlight %}


### Signing the pass and testing

Finally, build the Signpass Xcode project, and use the resulting executable to sign each pass bundle:

{% highlight csh %}
signpass -p MyPass.pass
{% endhighlight %}

This will create a `MyPass.pkpass` file. You can drag and drop the pkpass file in the iOS simulator for testing the pass.

At this point it's convenient to upload the pkpass file its public location in the web (the one specified by the barcode URL.) This way you can open Passcode in your iPhone and scan the barcode directly from the iOS simulator in your Mac screen.

The pass is ready to be distributed when it looks good on the simulator and on an actual device.


### Final notes

Business cards in Passbook are cool and a novelty, but we must keep in mind two things:

1. The Passbook application is available only on the iPhone and iPod touch. There's no Passbook on the iPad or the Mac, and obviously there's no Passbook in any non-Apple platform. This means that people with Android or Windows devices will only see garbage when downloading the pass file. It's a good idea to create a _vCard_ file for those who can't see Passbook files.

2. This is not Apple's intended use for Passbook, although there's no rule that prevents us from using Passbook in this way... for now.


Acknowledgment: I first read about Passbook business cards from [Justin Williams](http://carpeaqua.com/2014/05/30/find-me-wwdc-via-passbook/) just before the WWDC 2014. Thanks Justin!
