# HOWTO use SVGZ images on GitHub Pages

**TL;DR** use CloudFlare to modify HTTP request/response headers

## Introduction

[SVGZ](https://en.wikipedia.org/wiki/SVG#Compression) images are [SVG](https://en.wikipedia.org/wiki/SVG) images compressed with `gzip` algorithm. SVGZ images have `.svgz` suffix. Using SVGZ images on a web site (and on [GitHub Pages](https://pages.github.com/)) is problematic since the format requires a specific web server configuration. Here is why and how to use SVGZ image on GitHub Pages:

1. Both SVG and SVGZ images have the same `image/svg+xml` MIME type. See [MIME types on GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#mime-types-on-github-pages), and
2. since the MIME type has `compressible: true` attribute set, web servers like GitHub Pages double-compress SVGZ images with `gzip`. This confuses browsers which assume that a (once) uncompressed SVGZ image is just a SVG image. 

*SVG entry in MIME type DB used by GitHub Pages*
```json
"image/svg+xml": {
    "source": "iana",
    "compressible": true,
    "extensions": ["svg","svgz"]
```

[GitHub Pages](https://pages.github.com/) does not allow repo owners to specify content-compression rules per content type nor edit the response headers. Therefore, **it is impossible to use SVGZ images on [GitHub Pages](https://pages.github.com/)** without external help to modify the HTTP request/response headers.  

## Enter CloudFlare

[CloudFlare](cloudflare.com/) is a Content Distribution Network / CDN. It's free plan includes three [Transform Rules](https://developers.cloudflare.com/rules/transform/) to modify HTTP request/response headers. 

To use SVGZ images on [GitHub Pages](https://pages.github.com/), you need first to trick GitHub Pages not to double-compress SVGZ images and then add required HTTP response headers for the SVGZ images: 

1. Create a [CloudFlare](cloudflare.com/) account and add your web site there. You do not need to use CloudFlare Pages, just add your GitHub Pages as a "website".
2. Add a  HTTP Request Header Modification Rule to disable server content compression for SVGZ images:
* On CloudFlare dashboard: Rules -> Transform Rules -> Add _HTTP Request Header Modification_:
* A custom filter expression: `(ends_with(http.request.uri.path, ".svgz"))`
* `Set static`: `Accept-Encoding: gzip;q=0.0`

<img width="740" alt="image" src="https://user-images.githubusercontent.com/43722525/211169950-08b96a99-43f1-4a62-a4bc-e57749b6d029.png">

3. Add a  HTTP Response Header Modification Rule to add correct headers for SVGZ images:
* On CloudFlare dashboard: Rules -> Transform Rules -> Add _HTTP Response Header Modification_:
* A custom filter expression: `(ends_with(http.request.uri.path, ".svgz"))`
* `Set static`: `content-encoding: gzip`
* `Set static`: `content-type: image/svg+xml`. I am also setting `charset=utf-8` since my SVGZ content is UTF-8 encoded. 

<img width="729" alt="image" src="https://user-images.githubusercontent.com/43722525/211172546-a19d4000-7117-4b92-96b6-51f91a27683a.png">

4. You are all set! 

## Proof 

I have setup a [test page](https://blitzanalysiz.com/test.html). 

If you try to access the [test.svgz](https://github.com/Jylpah/jylpah.github.io/blob/main/test.svgz?raw=true) directly from GitHub, you will get an encoding error due to the double-compression. 

PS. I did try to first setup just a HTTP Response modification rule to declare double-encoding by:

```
Content-Encoding: gzip, gzip
```

This method is correct and described in [specification](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding#syntax). It is supported by all the major desktop browsers, but unfortunately iOS Safari (16.x at least) does not support it. 
