Cordova PayPal plugin
====================

# Overview #
paypal payment

[android, ios] [phonegap cli] [xdk] [cocoon] [phonegap build service]

requires paypal developer account https://developer.paypal.com/

# Change log #
```c
```

# Install plugin #

## Cordova cli ##
https://cordova.apache.org/docs/en/edge/guide_cli_index.md.html#The%20Command-Line%20Interface - npm install -g cordova@5.0.0
```c
cordova plugin add cordova-plugin-payment-paypal
(when build error, use github url: cordova plugin add https://github.com/cranberrygame/cordova-plugin-payment-paypal)
```

## Xdk ##
https://software.intel.com/en-us/intel-xdk - Download XDK - XDK PORJECTS - [specific project] - CORDOVA HYBRID MOBILE APP SETTINGS - Plugin Management - Add Plugins to this Project - Third Party Plugins -
```c
Plugin Source: Cordova plugin registry
Plugin ID: cordova-plugin-payment-paypal
```

## Cocoon ##
https://cocoon.io - Create project - [specific project] - Setting - Plugins - Custom - Git Url: https://github.com/cranberrygame/cordova-plugin-payment-paypal.git - INSTALL - Save<br>

## Phonegap build service (config.xml) ##
https://build.phonegap.com/ - Apps - [specific project] - Update code - Zip file including config.xml
```c
<gap:plugin name="cordova-plugin-payment-paypal" source="npm" />
```

## Construct2 ##
Download construct2 plugin<br>
https://dl.dropboxusercontent.com/u/186681453/pluginsforcordova/index.html<br>
How to install c2 native plugins in xdk, cocoon and cordova cli<br>
https://plus.google.com/102658703990850475314/posts/XS5jjEApJYV

# Server setting #
```c
//Get sandbox app client id (per app)
https://developer.paypal.com/ - Dashboard - My apps - [specific app] - Sandbox credentials - Client ID: YOUR_SANDBOX_APP_CLIENT_ID

//Get sandbox app secret (per app)
https://developer.paypal.com/ - Dashboard - My apps - [specific app] - Sandbox credentials - Secret: YOUR_SANDBOX_APP_SECRET

//Get live app client id (per app)
https://developer.paypal.com/ - Dashboard - My apps - [specific app] - Live credentials - Show - Click "Upgrade to a Premier or Business account" -... - Client ID: YOUR_LIVE_APP_CLIENT_ID

//Get live app secret (per app)
https://developer.paypal.com/ - Dashboard - My apps - [specific app] - Live credentials - Show - Click "Upgrade to a Premier or Business account" -... - Secret: YOUR_LIVE_APP_SECRET
```

# API #
```javascript

var sandboxClientId = this.properties[0];
var liveClientId = this.properties[1];
var merchantName = "My test shop";
var merchantPrivacyPolicyURL = "http://www.yourserver.com/privacypolicy.html";
var merchantUserAgreementURL = "http://www.yourserver.com/useragreement.html";
var sandboxMode = this.properties[2];		

document.addEventListener('deviceready', function(){
	var clientIDs = {"PayPalEnvironmentProduction": liveClientId, "PayPalEnvironmentSandbox": sandboxClientId};
	PayPalMobile['init'](clientIDs, function() {
		var config = new PayPalConfiguration({'merchantName': merchantName, 'merchantPrivacyPolicyURL': merchantPrivacyPolicyURL, 'merchantUserAgreementURL': merchantUserAgreementURL});
		PayPalMobile['prepareToRender'](sandboxMode==0?'PayPalEnvironmentProduction':'PayPalEnvironmentSandbox', config,
		function() {
		});				
	});
}, false);

Acts.prototype.BuyNow = function (amount, currency, shortDescription)
{
	if (!(this.runtime.isAndroid || this.runtime.isiOS))
		return;			
	if (typeof PayPalMobile == 'undefined')
		return;
	
	var self = this;

	// single payment
	//https://github.com/paypal/PayPal-Android-SDK#single-payment
	//https://github.com/paypal/PayPal-iOS-SDK#single-payment
	//payment's amount should match subtotal+shipping+tax. subtotal is also mandatory when paymentDetails are used. Hope it helps!
	//https://github.com/paypal/PayPal-Cordova-Plugin/issues/7
	//var paymentDetails = new PayPalPaymentDetails("50.00", "0.00", "0.00");
	//var payment = new PayPalPayment("50.00", "SGD", "Awesome Sauce", "Sale", paymentDetails);
	//var paymentDetails = new PayPalPaymentDetails(subtotal, shipping, tax);
	//var payment = new PayPalPayment(amount, currency, shortDescription, "Sale", paymentDetails);
	var payment = new PayPalPayment(amount, currency, shortDescription, "Sale");
	PayPalMobile['renderSinglePaymentUI'](payment,
	function(payment) {
		console.log("payment success: " + JSON.stringify(payment, null, 4));
		//payment['response']['state'] //approved
		//payment['response']['id'] //PAY-5UV89626C0126483AKRHCQJQ
		//payment['response']['create_time'] //2014-10-27T11:10:30Z
		//payment['response']['intent'] //sale			
		paymentId = payment['response']['id'];
		paymentCreateTime = payment['response']['create_time'];			
		paymentAmount = amount;
		paymentCurrency = currency;
		paymentShortDescription = shortDescription;		
		
		self.runtime.trigger(cr.plugins_.cranberrygame_CordovaPayPal.prototype.cnds.OnBuyNowSucceeded, self);			
	}, 
	function(error) {
		console.log(error);
		
		self.runtime.trigger(cr.plugins_.cranberrygame_CordovaPayPal.prototype.cnds.OnBuyNowFailed, self);			
	});		
};

Acts.prototype.BuyInFuture = function ()
{
	if (!(this.runtime.isAndroid || this.runtime.isiOS))
		return;			
	if (typeof PayPalMobile == 'undefined')
		return;
		
	var self = this;
	
	// future payment
	//https://github.com/paypal/PayPal-Android-SDK#future-payments
	//https://github.com/paypal/PayPal-iOS-SDK#future-payments
	PayPalMobile['renderFuturePaymentUI'](function(authorization) {
		console.log("authorization: " + JSON.stringify(authorization, null, 4));
		authorization['response']['code'] //ELqQrklopbu-7XQn_oqsy67oT6B9EHnBRvZHN5YIGWSyv7IcFm4EZ-3FLbmRuU_hui6hyfR7H2Elid3Nqn6qX8KHya4cLh1rAZweSUtcehnR-9nTlqYaae0lWQ5jJl1Qe-2Ll_B929dBWKmlK9kZyk
		
		self.runtime.trigger(cr.plugins_.cranberrygame_CordovaPayPal.prototype.cnds.OnBuyInFutureSucceeded, self);			
	},
	function(error) {
		console.log(error);
		
		self.runtime.trigger(cr.plugins_.cranberrygame_CordovaPayPal.prototype.cnds.OnBuyInFutureFailed, self);			
	});      		
};

Acts.prototype.ShareProfile = function ()
{
	if (!(this.runtime.isAndroid || this.runtime.isiOS))
		return;			
	if (typeof PayPalMobile == 'undefined')
		return;

	var self = this;
	
	// profile sharing
	//https://github.com/paypal/PayPal-Android-SDK#profile-sharing
	//https://github.com/paypal/PayPal-iOS-SDK#profile-sharing
	//https://developer.paypal.com/docs/integration/direct/identity/attributes/
	PayPalMobile['renderProfileSharingUI'](["profile", "email", "phone", "address", "futurepayments", "paypalattributes"], 
	function(authorization) {
		console.log("authorization: " + JSON.stringify(authorization, null, 4));
		
		self.runtime.trigger(cr.plugins_.cranberrygame_CordovaPayPal.prototype.cnds.OnShareProfileSucceeded, self);			
	}, 
	function(error) {
		console.log(error);
		
		self.runtime.trigger(cr.plugins_.cranberrygame_CordovaPayPal.prototype.cnds.OnShareProfileFailed, self);			
	}); 
};
	
```
# Examples #
<a href="https://github.com/cranberrygame/cordova-plugin-payment-paypal/blob/master/example/basic/index.html">example/basic/index.html</a><br>

# Test #

[![](http://img.youtube.com/vi/xXrVb8E8gMM/0.jpg)](https://www.youtube.com/watch?v=xXrVb8E8gMM&feature=youtu.be "Youtube")

You can also run following test apk.
https://dl.dropboxusercontent.com/u/186681453/pluginsforcordova/iap/apk.html

```c
//Create test account for sandbox
https://developer.paypal.com/ - Dashboard - Sandbox - Accounts - Create Account - Account type: Personal (buyer account) - ... - Create Account

Test with test account
```

# Useful links #

Cordova Plugins<br>
http://cranberrygame.github.io?referrer=github

Verify a mobile payment<br>
https://developer.paypal.com/webapps/developer/docs/integration/mobile/verify-mobile-payment/<br>
Make your first call<br>
https://developer.paypal.com/docs/integration/direct/make-your-first-call/<br>
Single Payment<br>
Receive immediate payment from a customer's PayPal account or payment card:<br>
Accept a Single Payment and receive back a proof of payment.<br>
On your server, Verify the Payment using PayPal's API.<br>
https://github.com/paypal/PayPal-Android-SDK#single-payment<br>
https://github.com/paypal/PayPal-iOS-SDK#single-payment

# Credits #

https://github.com/paypal/PayPal-Cordova-Plugin
