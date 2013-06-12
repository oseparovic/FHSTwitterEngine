FHSTwitterEngine
================

Original project created by [Nathaniel Symer](mailto:nate@natesymer.com), aka [@natesymer](http://twitter.com/natesymer)

`FHSTwitterEngine` can:

- Login through xAuth.
- Login through oAuth. Login UI based on [SA_OAuthTwitterEngineController](https://github.com/bengottlieb/Twitter-OAuth-iPhone)
- Make a request to every available API endpoints. Yes, even the legal ones.


Why `FHSTwitterEngine` is better than `MGTwitterEngine`:

- Lack of annoying delegates
- Does not send you to Dependency Hell over a JSON parser
- Less setup
- Synchronous allowing for easier implementation (see Usage)
- More implemented API endpoints
- Uses a fixed version of OAuthConsumer (mine)


**Setup**

1. Add the folder "FHSTwitterEngine" to your project and `#import "FHSTwitterEngine.h"` (I recommend `Prefix.pch`)
2. Link against `SystemConfiguration.framework`
3. In **Compile Sources** in the **Build Phases** tab of your target (e.g. MyCoolApp.app with the target icon next to it), add the `-fno-objc-arc` compiler flag to all files starting with `OA` (In other words, the `OAuthConsumer` library, *nothing* else)
4. Profit!!!!

**Usage:**

--> Create `FHSTwitterEngine` object:

    FHSTwitterEngine *engine = [[FHSTwitterEngine alloc]initWithConsumerKey:@"<consumer_key>" andSecret:@"<consumer_secret>"];
    
--> Login via OAuth:
    
    [engine showOAuthLoginControllerFromViewController:self];
    
--> Login via XAuth:
    
    dispatch_async(GCDBackgroundThread, ^{
    	@autoreleasepool {
    		NSError *error = [engine getXAuthAccessTokenForUsername:@"<username>" password:@"<password>"];
        	// Handle error
        	dispatch_sync(GCDMainThread, ^{
    			@autoreleasepool {
        			// Update UI
        		}
       		});
    	}
    });
    
--> Reload a saved access_token:

    [engine loadAccessToken];

--> End a session:

    [engine clearAccessToken];

--> Check if a session is valid:

    [engine isAuthorized];
    
--> Do an API call (POST):

    dispatch_async(GCDBackgroundThread, ^{
    	@autoreleasepool {
    		NSError *error = [engine twitterAPIMethod]; 
    		// Handle error
    		dispatch_sync(GCDMainThread, ^{
    			@autoreleasepool {
        			// Update UI
        		}
       		});
    	}
    });

--> Do an API call (GET):

    dispatch_async(GCDBackgroundThread, ^{
    	@autoreleasepool {
    		id twitterData = [engine twitterAPIMethod];
    		// Handle twitterData (see "About GET REquests")
    		dispatch_sync(GCDMainThread, ^{
    			@autoreleasepool {
        			// Update UI
        		}
       		});
    	}
    });

**Grand Central Dispatch**

So what are those `GCDBackgroundThread` and `GCDMainThread`?<br />
They are macros for `dispatch_async()` and `dispatch_sync()`, respectively. They make using GCD much easier. Yes, GCD is the designated way of adding async functionality to FHSTwitterEngine without losing the procedural paradigm.

**About POST requests**

All methods that send POST requests, including the xAuth login method, return `NSError`. If there is no error, they should return `nil`.

**General networking comments**

`FHSTwitterEngine` will attempt to preëmtively detect errors in your requests. This is designed to prevent flawed requests from being needlessly sent. This helps with rate limiting and networking turnover times.

**About GET requests**

GET methods return id. There returned object can be a member of one of the following classes:

- `NSDictionary`
- `NSArray`
- `UIImage`
- `NSString`
- `NSError`
- `nil`

In the case of `authenticatedUserIsBlocking:isID:`, an NSString will be returned. It will be `@"YES"` to indicate YES and `@"NO"` to indicate NO. Additionally, it will return an NSError if it fails. How else could I prevent false negatives?

**For the future**

Feel free to [email](mailto:nate@natesymer.com) me for suggestions.

- Specify AppID and Secret for each request for added security
- Mac support

**IMPORTANT**

`FHSTwitterEngine` contains an overhauled version of OAuthConsumer. The changes are:
- Removed `OADataFetcher`
- `OAAsynchronousDataFetcher` is now just a class method that takes arguments of a request and a block.
- Fixed string comparisons
- Fixed memory leaks
- Fixed bugs
- Compatibility with alternative versions of OAuthConsumer
