# url-metadata

Request a url and scrape the metadata from its HTML using Node.js or the browser. Under the hood, this package does some post-request processing on top of the javascript-native `fetch` API.

Includes:
- meta tags
- favicons
- [Open Graph Protocol (og:) Tags](http://ogp.me/)
- [Twitter Card Tags](https://developer.twitter.com/en/docs/twitter-for-websites/cards/overview/markup)
- [JSON-LD](https://moz.com/blog/json-ld-for-beginners)
- citations, per the Google Scholar spec
- h1-h6 tags
- automatic charset detection & decoding (optional)
- the full response body as a string of html (optional)

More details in the `Returns` section below.

To report a bug or request a feature please open an issue or pull request in [GitHub](https://github.com/laurengarcia/url-metadata). Please read the `Troublehsooting` section below *before* filing a bug.


## Usage
Works with Node.js version `>=18.0.0` or in the browser when bundled with Webpack or Browserify, etc. Use previous version `2.5.0` which uses the (now-deprecated) `request` module instead if you don't have access to javascript-native `fetch` API in your target environment.

Install in your project:
```
$ npm install url-metadata --save
```

In your project file:
```javascript
const urlMetadata = require('url-metadata');

const metadata = await urlMetadata(
  'https://www.npmjs.com/package/url-metadata',
  {
    includeResponseBody: true,
    ensureSecureImageRequest: true
  }
);
console.log('fetched metadata:', metadata)
```

### Options & Defaults
The default options are the values below. To override the default options, pass in a second options argument.
```javascript
const options = {
  // custom request headers
  requestHeaders: {
    'User-Agent': 'url-metadata/3.0 (npm module)',
    'From': 'example@example.com',
  }

  // `fetch` API cache setting for request
  cache: 'no-cache',

  // `fetch` API mode (ex: 'cors', 'no-cors', 'same-origin', etc)
  mode: 'cors',

  // charset to decode response body with
  // (ex: 'auto', 'utf-8', 'windows-1251')
  // defaults to auto-detect charset based on `Content-Type` header or meta tag
  // if no charset is detected, the default `auto` option decodes with `utf-8`
  // override by passing in charset to decode with (ex: 'windows-1251')
  decode: 'auto',

  // timeout in milliseconds, default is 10 seconds
  timeout: 10000,

  // number of characters to truncate description to
  descriptionLength: 750,

  // force image urls in selected tags to use https,
  // valid for 'image', 'og:image', 'og:image:secure_url'
  // tags & favicons with full paths
  ensureSecureImageRequest: true,

  // return raw response body as string
  includeResponseBody: false
};

const metadata = await urlMetadata(
  'https://www.npmjs.com/package/url-metadata',
  options
);
```

### Returns
Returns a promise that is resolved with an object if the response is successful. Note that the `url` field returned will be the last hop in the request chain. So if you passed in a url that was generated by a url shortener you'll get back the final destination as the `url`.

When the `decode` option is set to `auto` (the default), this module will infer the character set to decode the metadata with from the `Content-Type` headers or meta tags, per html5 spec.

The returned `metadata` object consists of key/value pairs that are all strings, with a few exceptions:
- `favicons` returns as array of objects containing key/value pairs (strings)
- `jsonld` returns as object containing key/value pairs (strings)
- if a meta tag named `og:type` with `content="article"` is present we return an object named `article` containing key/value pairs (strings)
- all meta tags that begin with `citation_` (ex: `citation_author`) return with keys as strings and values that are an array of strings to conform to the [Google Scholar spec](https://www.google.com/intl/en/scholar/inclusion.html#indexing) which allows for multiple citation meta tags with different content values. So if the html contains:
```
<meta name="citation_author" content="Arlitsch, Kenning">
<meta name="citation_author" content="OBrien, Patrick">
```
... this module will return:
```
'citation_author': ["Arlitsch, Kenning", "OBrien, Patrick"],
```

### Troubleshooting

**Issue:** Response status code `0` and/ or `CORS` errors
This is coming directly from the javascript-native `fetch` API used by this package. The request failed at either the network or protocol level. Possible causes:
- CORS errors. Try changing the mode option (ex: `cors`, `no-cors`, `same-origin`, etc) or setting the `Access-Control-Allow-Origin` header on the server response from the url you are requesting if you have access to it.
- A browser plugin such as an ad-blocker or privacy protector blocking the request.
- Could also be caused by trying to access an `https` resource that has an invalid certificate, or trying to access an `http` resource from a page with an `https` origin.

**Issue:** `fetch is not defined`
You're either in a Node.js or browser environment that doesn't have javascript's `fetch` method available. Try upgrading your environment (Node.js version `>=18.0.0`), or you can use an earlier version of this package (version 2.5.0).
