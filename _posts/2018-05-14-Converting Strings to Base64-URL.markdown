---
layout: post
title:  "Converting Strings to Base64-URL"
date:   2018-05-14 20:59:30
categories: [ swift, google, jwt, server ]
---

I've been working with Google's GSuite Directory API in a Swift app running on Ubuntu. While Google has an SDK for using Swift with iOS and Mac apps, they don't supply anything for Linux. Maybe there would be a way of getting it to work, but I would rather not use a huge SDK for some simple calls to the Directory API.

Authenticating gets a bit hairy, though when you're not using Google's SDKs. The first step is to set up a "Service Account" which you can give permission to access admin controls for your GSuite domain. Google then gives you a private key that you can use to get API bearer tokens (which are each valid for an hour). In order to get that token, you have to construct and submit a JWT (or JSON Web Token, pronounced "jot" according to Google).

I'll get into some other specifics of putting this together at another date, but the first thing you'll need to figure out is how to convert some Swift strings to Base64-URL. If you're not familiar, Base64-URL is a variant of base 64 text encoding that is safe to use for URLS. That means replacing the characters "+" and "/", with "-" and "_". In addition, regular base 64 can include padding at the end in the form of one or two "=" characters, to make sure that the final digit has at 6-bits of data. These need to be stripped.

Below is code adapted from Github user [Carterhouse] found in [this repo]. He wrote it as an extension to Swift's Data type, but for my purposes I found extending String more practical.

In a future post I'll detail more steps in building 

{% highlight swift %}
extension String {
    
    func fromBase64URL() -> String? {
	    var base64 = self
        base64 = base64.replacingOccurrences(of: "-", with: "+")
        base64 = base64.replacingOccurrences(of: "_", with: "/")
        while base64.count % 4 != 0 {
            base64 = base64.appending("=")
        }
        guard let data = Data(base64Encoded: base64) else {
	        return nil
        }
        return String(data: data, encoding: .utf8)
    }
    
    func toBase64URL() -> String {
	    var result = Data(self.utf8).base64EncodedString()
        result = result.replacingOccurrences(of: "+", with: "-")
        result = result.replacingOccurrences(of: "/", with: "_")
        result = result.replacingOccurrences(of: "=", with: "")
        return result
    }
}
{% endhighlight %}

It's also worth noting that if you are using the [Perfect] toolkit for building server-side Swift apps, they provide an API for creating a JWT in their [Crypto] module, and it should handle the conversion for you. If you're using Perfect already and need to call the Google Directory API, not much reason not to use it.

[Carterhouse]: https://github.com/Charterhouse
[this repo]: https://github.com/Charterhouse/base64url-swift
[Perfect]: https://www.perfect.org
[Crypto]: https://www.perfect.org/docs/crypto.html


