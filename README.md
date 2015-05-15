**SOAPEngine**
================

This generic `SOAP` client allows you to access web services using a your `iOS` app and `Mac OS X` app.

With this Framework you can create iPhone, iPad and Mac OS X apps that supports SOAP Client Protocol. This framework able executes methods at remote web services with SOAP standard protocol.

## Features
* Support both 2001 (v1.1) and 2003 (v1.2) `XML` schema.
* Support array, array of structs, dictionary and sets.
* Support for user-defined object with serialization of complex data types and array of complex data types, even embedded multilevel structures.
* Supports `ASMX` Services, `WCF` Services (`svc`) and now also the `WSDL` definitions.
* Supports Basic Authentication, `WS-Security`, Client side Certificate and custom security header.
* Supports iOS Social Account to send OAuth 2.0 token on the request.
* `AES256` or `3DES` Encrypt/Decrypt data without SSL security.
* An example of service and how to use it is included in source code.

## Requirements for iOS
* iOS 5.1.1, and later
* `XCode` 5.0 or later
* Security.framework
* Accounts.framework
* Foundation.framework
* UIKit.framework
* libxml2.dylib

## Requirements for Mac OS X
* OS X 10.9 and later
* `XCode` 5.0 or later
* Security.framework
* Accounts.framework
* Foundation.framework
* AppKit.framework
* Cocoa.framework
* libxml2.dylib

## Limitations
* for `WCF` services, only supports basic http bindings (`<basicHttpBinding>`).
* in `Mac OS X` unsupported image objects (instead you can use the `NSData`).

## How to use

with delegates :

``` objective-c
	#import <SOAPEngine/SOAPEngine.h>

	// standard soap service (.asmx)
	SOAPEngine *soap = [[SOAPEngine alloc] init];
	soap.userAgent = @"SOAPEngine";
	soap.delegate = self; // use SOAPEngineDelegate

	// each single value
	[soap setValue:@"my-value1" forKey:@"Param1"];
	[soap setIntegerValue:1234 forKey:@"Param2"];
	// service url without ?WSDL, and you can search the soapAction in the WSDL
	[soap requestURL:@"http://www.my-web.com/my-service.asmx" 
		  soapAction:@"http://www.my-web.com/My-Method-name"];
 
	#pragma mark - SOAPEngine Delegates

	- (void)soapEngine:(SOAPEngine *)soapEngine didFinishLoading:(NSString *)stringXML {

	        NSDictionary *result = [soapEngine dictionaryValue];
        	// read data from a dataset table
        	NSArray *list = [result valueForKeyPath:@"NewDataSet.Table"];
	}
```

with block programming :

``` objective-c
	#import <SOAPEngine/SOAPEngine.h>
	
	// TODO: your user object
	MyClass myObject = [[MyClass alloc] init];
	
	SOAPEngine *soap = [[SOAPEngine alloc] init];
	soap.userAgent = @"SOAPEngine";
	soap.version = VERSION_WCF_1_1; // WCF service (.svc)
	
	// service url without ?WSDL, and you can search the soapAction in the WSDL
	[soap requestURL:@"http://www.my-web.com/my-service.svc"
		  soapAction:@"http://www.my-web.com/my-interface/my-method"
			   value:myObject
			complete:^(NSInteger statusCode, NSString *stringXML) {
		    	NSDictionary *result = [soap dictionaryValue];
				NSLog(@"%@", result);
			} failWithError:^(NSError *error) {
				NSLog(@"%@", error);
			}];
```	

directly from WSDL (not recommended is slow) :

``` objective-c
	#import <SOAPEngine/SOAPEngine.h>
	
	// TODO: your user object
	MyClass myObject = [[MyClass alloc] init];
	
	SOAPEngine *soap = [[SOAPEngine alloc] init];
	soap.userAgent = @"SOAPEngine";
	
	// service url with WSDL, and operation (method name) without tempuri
	[soap requestWSDL:@"http://www.my-web.com/my-service.amsx?wsdl"
		    operation:@"my-method-name"
			    value:myObject
			completeWithDictionary:^(NSInteger statusCode, NSDictionary *dict) {

              NSLog(@"Result: %@", dict);

			} failWithError:^(NSError *error) {

				NSLog(@"%@", error);
			}];
```	

with notifications :

``` objective-c
	#import <SOAPEngine/SOAPEngine.h>

	// TODO: your user object
	MyClass myObject = [[MyClass alloc] init];
	
	SOAPEngine *soap = [[SOAPEngine alloc] init];
	soap.userAgent = @"SOAPEngine";
	soap.version = VERSION_WCF_1_1; // WCF service (.svc)
		
    [[NSNotificationCenter defaultCenter] 
    			addObserver:self 
    			   selector:@selector(soapEngineDidFinishLoading:) 
    				   name:SOAPEngineDidFinishLoadingNotification 
    				 object:nil];
	
	// service url without ?WSDL, and you can search the soapAction in the WSDL
	[soap requestURL:@"http://www.my-web.com/my-service.svc" 
		  soapAction:@"http://www.my-web.com/my-interface/my-method"
		  	   value:myObject];
	
	#pragma mark - SOAPEngine Notifications
	
	- (void)soapEngineDidFinishLoading:(NSNotification*)notification
	{
    	SOAPEngine *engine = notification.object; // SOAPEngine object
    	NSDictionary *result = [engine dictionaryValue];
    	NSLog(@"%@", result);
	}
```

Synchronous request :

``` objective-c
	#import <SOAPEngine/SOAPEngine.h>
	
	NSError *error = nil;
    SOAPEngine *soap = [[SOAPEngine alloc] init];
    soap.responseHeader = YES; // use only for non standard MS-SOAP service like PHP
    NSDictionary *dict = [soap syncRequestURL:@"http://www.my-web.com/my-service.amsx" 
    						soapAction:@"http://tempuri.org/my-method" error:&error];
    NSLog(@"error: %@, result: %@", error, dict)
```

Swift language :

``` swift
        var soap = SOAPEngine()
        soap.userAgent = "SOAPEngine"
        soap.actionNamespaceSlash = true
        soap.version = VERSION_1_1
        soap.responseHeader = true // use only for non standard MS-SOAP service
        
        soap.setValue("param-value", forKey: "param-name")
        soap.requestURL("http://www.my-web.com/my-service.asmx",
            soapAction: "http://www.my-web.com/My-Method-name",
            completeWithDictionary: { (statusCode : Int, 
            					 dict : [NSObject : AnyObject]!) -> Void in
                
                var result:Dictionary = dict as Dictionary
                NSLog("%@", result)
                
            }) { (error : NSError!) -> Void in
                
                NSLog("%@", error)
        }
```
	
settings for soap authentication :

``` objective-c
	#import <SOAPEngine/SOAPEngine.h>

	SOAPEngine *soap = [[SOAPEngine alloc] init];
	soap.userAgent = @"SOAPEngine";
	
	// authorization
	soap.authorizationMethod = SOAP_AUTH_BASIC; // basic auth
	soap.username = @"my-username";
	soap.password = @"my-password";
	
	// TODO: your code here...
	
```	

settings for Social OAuth 2.0 token :

``` objective-c
	#import <SOAPEngine/SOAPEngine.h>
	#import <Accounts/Accounts.h>

	SOAPEngine *soap = [[SOAPEngine alloc] init];
	soap.userAgent = @"SOAPEngine";
	
	// token authorization
	soap.authorizationMethod = SOAP_AUTH_SOCIAL;
	soap.apiKey = @"1234567890"; // your apikey https://dev.twitter.com/
	soap.socialName = ACAccountTypeIdentifierTwitter;
	
	// TODO: your code here...
	
```	

encryption/decryption data without SSL/HTTPS :

``` objective-c
	#import <SOAPEngine/SOAPEngine.h>

	SOAPEngine *soap = [[SOAPEngine alloc] init];
	soap.userAgent = @"SOAPEngine";
	soap.encryptionType = SOAP_ENCRYPT_AES256; // or SOAP_ENCRYPT_3DES
	soap.encryptionPassword = @"my-password";

	// TODO: your code here...
	
```	
W3Schools example :

``` objective-c
	SOAPEngine *soap = [[SOAPEngine alloc] init];
    soap.actionNamespaceSlash = YES;

    // w3schools Celsius to Fahrenheit
    [soap setValue:@"30" forKey:@"Celsius"];
    [soap requestURL:@"http://www.w3schools.com/webservices/tempconvert.asmx"  
        soapAction:@"http://www.w3schools.com/webservices/CelsiusToFahrenheit" 
        complete:^(NSInteger statusCode, NSString *stringXML) {

        NSLog(@"Result: %f", [soap floatValue]);

    } failWithError:^(NSError *error) {

        NSLog(@"%@", error);
    }];
	
```	

WebServiceX example :

``` objective-c
	SOAPEngine *soap = [[SOAPEngine alloc] init];
    soap.actionNamespaceSlash = NO;

    [soap setValue:@"Roma" forKey:@"CityName"];
    [soap setValue:@"Italy" forKey:@"CountryName"];
    [soap requestURL:@"http://www.webservicex.com/globalweather.asmx"
          soapAction:@"http://www.webserviceX.NET/GetWeather"
          completeWithDictionary:^(NSInteger statusCode, NSDictionary *dict) {
              
              NSLog(@"Result: %@", dict);
              
          } failWithError:^(NSError *error) {
    
              NSLog(@"%@", error);
          }];
          	
```	

PAYPAL example :

``` objective-c
	SOAPEngine *soap = [[SOAPEngine alloc] init];

    // PAYPAL associates a set of API credentials with a specific PayPal account
    // you can generate credentials from this https://developer.paypal.com/docs/classic/api/apiCredentials/
    // and convert to a p12 from terminal use :
    // openssl pkcs12 -export -in cert_key_pem.txt -inkey cert_key_pem.txt -out paypal_cert.p12
    soap.authorizationMethod = SOAP_AUTH_PAYPAL;
    soap.username = @"support_api1.your-username";
    soap.password = @"your-api-password";
    soap.clientCerficateName = @"paypal_cert.p12";
    soap.clientCertificatePassword = @"certificate-password";
    soap.responseHeader = YES;
    // use paypal for urn:ebay:api:PayPalAPI namespace
    [soap setValue:@"0" forKey:@"paypal:ReturnAllCurrencies"];
    // use paypal1 for urn:ebay:apis:eBLBaseComponents namespace
    [soap setValue:@"119.0" forKey:@"paypal1:Version"]; // ns:Version in WSDL file
    // certificate : https://api.paypal.com/2.0/ sandbox https://api.sandbox.paypal.com/2.0/
    // signature : https://api-3t.paypal.com/2.0/ sandbox https://api-3t.sandbox.paypal.com/2.0/
    [soap requestURL:@"https://api.paypal.com/2.0/"
          soapAction:@"GetBalance" completeWithDictionary:^(NSInteger statusCode, NSDictionary *dict) {
          
        NSLog(@"Result: %@", dict);
        
    } failWithError:^(NSError *error) {
    
        NSLog(@"%@", error);
    }];
          	
```	

Upload file :

``` objective-c
	SOAPEngine *soap = [[SOAPEngine alloc] init];

	// read local file
    NSData *data = [NSData dataWithContentsOfFile:@"my_video.mp4"];

	// send file data
    [soap setValue:data forKey:@"video"];
    [soap requestURL:@"http://www.my-web.com/my-service.asmx"
          soapAction:@"http://www.my-web.com/UploadFile"
          completeWithDictionary:^(NSInteger statusCode, NSDictionary *dict) {
              
              NSLog(@"Result: %@", dict);
              
          } failWithError:^(NSError *error) {
    
              NSLog(@"%@", error);
          }];
          	
```	

Download file :

``` objective-c
	SOAPEngine *soap = [[SOAPEngine alloc] init];

	// send filename to remote webservice
    [soap setValue:"my_video.mp4" forKey:@"filename"];
    [soap requestURL:@"http://www.my-web.com/my-service.asmx"
          soapAction:@"http://www.my-web.com/DownloadFile"
          completeWithDictionary:^(NSInteger statusCode, NSDictionary *dict) {
            
            // local writable directory
			NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
			NSString *filePath = [[paths firstObject] stringByAppendingPathComponent:@"my_video.mp4"];

			// the service returns file data in the tag named video
			NSData *data = dict[@"video"];
		    [data writeToFile:@"my_video.mp4" atomically:YES];
              
          } failWithError:^(NSError *error) {
    
              NSLog(@"%@", error);
          }];
          	
```	

## Optimizations
When using the method named requestWSDL three steps are performed : 

1. retrieve the WSDL with an http request.
2. processing to identify the soapAction.
3. calls the method with an http request.

this is not optimized, very slow, instead you can use the optimization below : 

1. retrieving manually the SOAPAction directly from WSDL (once with your favorite browser).
2. use the method named requestURL instead of requestWSDL.

## Install in your apps

1. add -lxml2 in Build Settings --> Other Linker Flags.
![Other Linker Flags](/screen/otherlinkerflags.png)

2. add /usr/include/libxml2 in Build Settings --> Header Search Paths.
![Header Search Paths](/screen/headersearchpaths.png)

3. SOAPEngine64.framework (for iOS apps) or SOAPEngineOSX.framework (for Mac OS X apps).
4. add Security.framework.
5. add Accounts.framework.
6. add AppKit.framework (only for Mac OS X apps, not required for iOS apps).
![Frameworks](/screen/frameworks.png)

7. in your class import <SOAPEngine64/SOAPEngine.h> or <SOAPEngineOSX/SOAPEngine.h>.
![import](/screen/codeimport.png)

**[GET IT NOW!](http://www.prioregroup.com/iphone/soapengine.aspx)**

##Contacts

- https://twitter.com/DaniloPriore
- https://www.facebook.com/prioregroup
- http://www.prioregroup.com/
- http://it.linkedin.com/in/priore/
