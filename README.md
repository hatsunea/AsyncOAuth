AsyncOAuth
==========

Portable Client Library and HttpClient based OAuth library, including all platform(for PCL as .NET 4.0, .NET 4.5, Silverlight4, Silverlight5, Windows Phone 7.5, Windows Phone 8.0, Windows Store Apps).

Install
---
using with NuGet(Including PreRelease), [AsyncOAuth](https://nuget.org/packages/AsyncOAuth/)
```
PM> Install-Package AsyncOAuth -Pre
```

Usage
---
at first, you must initialize hash function(ApplicationStart etc...)

```csharp
// Silverlight, Windows Phone, Console, Web, etc...
OAuthUtility.ComputeHash = (key, buffer) => { using (var hmac = new HMACSHA1(key)) { return hmac.ComputeHash(buffer); } };
 
// Windows Store Apps
AsyncOAuth.OAuthUtility.ComputeHash = (key, buffer) =>
{
    var crypt = Windows.Security.Cryptography.Core.MacAlgorithmProvider.OpenAlgorithm("HMAC_SHA1");
    var keyBuffer = Windows.Security.Cryptography.CryptographicBuffer.CreateFromByteArray(key);
    var cryptKey = crypt.CreateKey(keyBuffer);
 
    var dataBuffer = Windows.Security.Cryptography.CryptographicBuffer.CreateFromByteArray(buffer);
    var signBuffer = Windows.Security.Cryptography.Core.CryptographicEngine.Sign(cryptKey, dataBuffer);
 
    byte[] value;
    Windows.Security.Cryptography.CryptographicBuffer.CopyToByteArray(signBuffer, out value);
    return value;
};
```

Create HttpClient with OAuthMessageHandler

```csharp
var client = new HttpClient(new OAuthMessageHandler("consumerKey", "consumerSecret", new AccessToken("accessToken", "accessTokenSecret")));
 
// shorthand(result is same above)
var client = OAuthUtility.CreateOAuthClient("consumerKey", "consumerSecret", new AccessToken("accessToken", "accessTokenSecret"));
```

operation same as HttpClient
```csharp
// Get
var json = await client.GetStringAsync("http://api.twitter.com/1.1/statuses/home_timeline.json?count=" + count + "&page=" + page);
 
// Post
var content = new FormUrlEncodedContent(new[] { new KeyValuePair<string, string>("status", status) });
var response = await client.PostAsync("http://api.twitter.com/1.1/statuses/update.json", content);
var json = await response.Content.ReadAsStringAsync();
 
// Multi Post
var content = new MultipartFormDataContent();
content.Add(new StringContent(status), "\"status\"");
content.Add(new ByteArrayContent(media), "media[]", "\"" + fileName + "\"");
 
var response = await client.PostAsync("https://upload.twitter.com/1/statuses/update_with_media.json", content);
var json = await response.Content.ReadAsStringAsync();
```

Sample
---
more sample, please see AsyncOAuth.ConsoleApp(Twitter.cs, Hatena.cs), AsyncOAuth.WindowsStoreApp  
sample contains authorize flow

```csharp
// sample flow for Twitter authroize
public async static Task<AccessToken> AuthorizeSample(string consumerKey, string consumerSecret)
{
    // create authorizer
    var authorizer = new OAuthAuthorizer(consumerKey, consumerSecret);

    // get request token
    var tokenResponse = await authorizer.GetRequestToken("https://api.twitter.com/oauth/request_token");
    var requestToken = tokenResponse.Token;

    var pinRequestUrl = authorizer.BuildAuthorizeUrl("https://api.twitter.com/oauth/authorize", requestToken);

    // open browser and get PIN Code
    Process.Start(pinRequestUrl);

    // enter pin
    Console.WriteLine("ENTER PIN");
    var pinCode = Console.ReadLine();

    // get access token
    var accessTokenResponse = await authorizer.GetAccessToken("https://api.twitter.com/oauth/access_token", requestToken, pinCode);

    // save access token.
    var accessToken = accessTokenResponse.Token;
    Console.WriteLine("Key:" + accessToken.Key);
    Console.WriteLine("Secret:" + accessToken.Secret);

    return accessToken;
}
```

License
---
under [MIT License](http://opensource.org/licenses/MIT)
