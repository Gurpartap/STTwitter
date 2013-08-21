## STTwitter

_A comprehensive Objective-C library for Twitter REST API 1.1_

> STTwitter will be presented at [SoftShake](http://soft-shake.ch) 2013, Geneva, on October 24th.

1. [Installation](#installation)
2. [Code Snippets](#code-snippets)
3. [Various Kinds of OAuth Connections](#various-kinds-of-oauth-connections)
4. [OAuth Consumer Tokens](#oauth-consumer-tokens)
5. [Demo / Test Project](#demo--test-project)
6. [Troubleshooting](#troubleshooting)
7. [Developers](#developers)
8. [Applications Using STTwitter](#applications-using-sttwitter)
9. [BSD 3-Clause License](#bsd-3-clause-license)

### Installation

Drag and drop STTwitter directory into your project and that's it.

If you want to use CocoaPods, include the STTwitter pod in your project's podfile and install it:

    pod 'STTwitter'
    pod install

Note that STTwitter requires iOS 5+ or OS X 10.7+.

### Code Snippets

Notes:

- STTwitter must be used from the main thread
- all callbacks are called on the main thread

##### Instantiate STTwitterAPI

    STTwitterAPI *twitter = [STTwitterAPI twitterAPIWithOAuthConsumerName:@""
                                                              consumerKey:@""
                                                           consumerSecret:@""
                                                                 username:@""
                                                                 password:@""];

##### Verify the credentials

    [twitter verifyCredentialsWithSuccessBlock:^(NSString *username) {
        // ...
    } errorBlock:^(NSError *error) {
        // ...
    }];

##### Get the timeline statuses

    [twitter getHomeTimelineSinceID:nil
                              count:100
                       successBlock:^(NSArray *statuses) {
        // ...
    } errorBlock:^(NSError *error) {
        // ...
    }];

##### Streaming API

    [twitter getStatusesSampleDelimited:nil
                          stallWarnings:nil
                          progressBlock:^(id response) {
        // ...
    } stallWarningBlock:nil
             errorBlock:^(NSError *error) {
        // ...
    }];

##### App Only Authentication

    STTwitterAPI *twitter = [STTwitterAPI twitterAPIAppOnlyWithConsumerKey:@""
                                                            consumerSecret:@""];
    
    [twitter verifyCredentialsWithSuccessBlock:^(NSString *bearerToken) {
        
        [twitter getUserTimelineWithScreenName:@"barackobama"
                                  successBlock:^(NSArray *statuses) {
            // ...
        } errorBlock:^(NSError *error) {
            // ...
        }];
    
    } errorBlock:^(NSError *error) {
        // ...
    }];

### Various Kinds of OAuth Connections

You can instantiate `STTwitterAPI` in three ways:

- use the Twitter account set in OS X Preferences or iOS Settings
- use a custom `consumer key` and `consumer secret` (three flavors)
  - get an URL, fetch a PIN, enter it in your app, get oauth access tokens  
  - set `username` and `password`, get oauth access tokens with XAuth, if the app is entitled to
  - set `oauth token` and `oauth token secret` directly
- use the [Application Only](https://dev.twitter.com/docs/auth/application-only-auth) authentication and get / use a "bearer token"

So there are five cases altogether, hence these five methods:

    + (STTwitterAPI *)twitterAPIOSWithFirstAccount;

    + (STTwitterAPI *)twitterAPIWithOAuthConsumerKey:(NSString *)consumerKey
                                      consumerSecret:(NSString *)consumerSecret;

    + (STTwitterAPI *)twitterAPIWithOAuthConsumerKey:(NSString *)consumerKey
                                      consumerSecret:(NSString *)consumerSecret
                                            username:(NSString *)username
                                            password:(NSString *)password;

    + (STTwitterAPI *)twitterAPIWithOAuthConsumerKey:(NSString *)consumerKey
                                      consumerSecret:(NSString *)consumerSecret
                                          oauthToken:(NSString *)oauthToken
                                    oauthTokenSecret:(NSString *)oauthTokenSecret;
                   
    + (STTwitterAPI *)twitterAPIAppOnlyWithConsumerKey:(NSString *)consumerKey
                                        consumerSecret:(NSString *)consumerSecret;

##### Reverse Authentication

Reference: [https://dev.twitter.com/docs/ios/using-reverse-auth](https://dev.twitter.com/docs/ios/using-reverse-auth)

The most common use case of reverse authentication is letting users register/login to a remote service with their OS X or iOS Twitter account.

    iOS/OSX     Twitter     Server
    -------------->                 reverse auth.
    < - - - - - - -                 access tokens
        
    ----------------------------->  access tokens
        
                   <--------------  access Twitter on user's behalf
                    - - - - - - ->

Here is how to use reverse authentication with STTwitter:

    STTwitterAPI *twitter = [STTwitterAPI twitterAPIWithOAuthConsumerName:nil
                                                              consumerKey:@"CONSUMER_KEY"
                                                           consumerSecret:@"CONSUMER_SECRET"];
    
    [twitter postReverseOAuthTokenRequest:^(NSString *authenticationHeader) {
        
        STTwitterAPI *twitterAPIOS = [STTwitterAPI twitterAPIOSWithFirstAccount];
        
        [twitterAPIOS verifyCredentialsWithSuccessBlock:^(NSString *username) {
            
            [twitterAPIOS postReverseAuthAccessTokenWithAuthenticationHeader:authenticationHeader
                                                                successBlock:^(NSString *oAuthToken,
                                                                               NSString *oAuthTokenSecret,
                                                                               NSString *userID,
                                                                               NSString *screenName) {
                                                                    
                                                                    // use the tokens...
                                                                    
                                                                } errorBlock:^(NSError *error) {
                                                                    // ...
                                                                }];
            
        } errorBlock:^(NSError *error) {
            // ...
        }];
        
    } errorBlock:^(NSError *error) {
        // ...
    }];

### OAuth Consumer Tokens

In Twitter REST API v1.1, each client application must authenticate itself with `consumer key` and `consumer secret` tokens. You can request consumer tokens for your app on Twitter website: [https://dev.twitter.com/apps](https://dev.twitter.com/apps).

STTwitter demo project comes with `TwitterClients.plist` where you can enter your own consumer tokens.

### Demo / Test Project

There is a demo project for OS X in `demo_osx`, which lets you choose how to get the OAuth tokens (see below).

Once you got the OAuth tokens, you can get your timeline and post a new status.

There is also a simple iOS demo project in `demo_ios`.

![STTwitter](Art/STTwitter.png "STTwitter Demo OS X") 
<img border="1" src="Art/ios.png" alt="STTwitter Demo iOS"></img> 
<img border="1" src="Art/STTweet.png" alt="STTwitter Demo Tweet"></img> 

### Troubleshooting

##### xAuth

Twitter restricts the xAuth authentication process to xAuth-enabled consumer tokens only. So if you get an error like `NSURLErrorDomain Code=-1012, Unhandled authentication challenge type - NSURLAuthenticationMethodOAuth2` while accessing `https://api.twitter.com/oauth/access_token` then your consumer tokens are probably not xAuth-enabled. You can read more on this on Twitter website [https://dev.twitter.com/docs/oauth/xauth](https://dev.twitter.com/docs/oauth/xauth) and ask Twitter to enable the xAuth authentication process for your consumer tokens.

##### Anything Else

Please [fill an issue](https://github.com/nst/STTwitter/issues) on GitHub.

### Developers

The application only interacts with `STTwitterAPI`.

`STTwitterAPI` maps Objective-C methods with all documented Twitter API endpoints.

You can create your own convenience methods with fewer parameters. You can also use this generic methods directly:

	- (void)fetchResource:(NSString *)resource
	           HTTPMethod:(NSString *)HTTPMethod
	        baseURLString:(NSString *)baseURLString
	           parameters:(NSDictionary *)params
	        progressBlock:(void(^)(id json))progressBlock
	         successBlock:(void(^)(id json))successBlock
	           errorBlock:(void(^)(NSError *error))errorBlock;
           
##### Layer Model
     
     +-------------------------------------------------+
     |                Your Application                 |
     +-------------------------------------------------+
                             v
     +-------------------------------------------------+
     |                  STTwitterAPI                   |
     +-------------------------------------------------+
            v              v                 v
     + - - - - - - - - - - - - - - - - - - - - - - - - +
     |              STTwitterOAuthProtocol             |
     + - - - - - - - - - - - - - - - - - - - - - - - - +
            v              v                 v
     +-------------+----------------+------------------+
     | STTwitterOS | STTwitterOAuth | STTwitterAppOnly |
     |             +----------------+------------------+
     |             |          STHTTPRequest            |
     +-------------+-----------------------------------+
      |
      + Accounts.framework
      + Social.framework
     
##### Summary
     
     * STTwitterAPI
        - can be instantiated with the authentication mode you want
        - provides methods to interact with each Twitter API endpoint

     * STTwitterOAuthProtocol
        - provides generic methods to POST and GET resources on Twitter hosts
     
     * STTwitterOS
        - uses Twitter accounts defined in OS X Preferences or iOS Settings
        - uses OS X / iOS frameworks to interact with Twitter API
     
     * STTwitterOAuth
        - implements OAuth and xAuth authentication

     * STTwitterAppOnly
        - implements the 'app only' authentication
        - https://dev.twitter.com/docs/auth/application-only-auth

     * STHTTPRequest
        - block-based wrapper around NSURLConnection
        - https://github.com/nst/STHTTPRequest

### Applications Using STTwitter

* [Adium](http://adium.im/) developers have [chosen](http://permalink.gmane.org/gmane.network.instant-messaging.adium.devel/2332) to use STTwitter to handle Twitter connectivity in Adium, starting from version 1.5.7.

### BSD 3-Clause License

Copyright (c) 2012-2013, Nicolas Seriot
All rights reserved.
    
Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:
    
* Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
* Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.
* Neither the name of the Nicolas Seriot nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.
    
THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
