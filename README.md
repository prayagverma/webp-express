# WebP Express

Serve autogenerated WebP images instead of jpeg/png to browsers that supports WebP. Works on anything (media library, galleries, theme images etc).

## Description

This plugin let's you take advantage of the WebP image format with only a little effort. Install, configure, test, forget - and enjoy the increased performance of your website.

The plugin basically routes jpeg/png images to an image converter, or - if the image converter has already converted the image - directly to a converted image. The approach has the benefit that is works regardless of how an image found its way into your server - be it Media Library, Galleries, or even theme images referenced with CSS.

The plugin builds on [WebPConvert](https://github.com/rosell-dk/webp-convert) and some of the ecosystem around it:   [WebPConvertAndServe](https://github.com/rosell-dk/webp-convert-and-serve) and [WebPOnDemand](https://github.com/rosell-dk/webp-on-demand)

#### Benefits
- Much faster load time on images in Chrome browsers. The converted images are typically less than half the size (for jpeg), while maintaining the same quality. Bear in mind that images typically are responsible for most of the bandwidth usage.
- Better user experience (whether performance goes from terrible to bad, or from good to impressive, it is a benefit)
- Better ranking in Google searches (performance is taken into account by Google)
- Less bandwidth consumption - makes a difference when abroad and in the parts of the world with slow and expensive internet connections.


## Installation

1. Upload the plugin files to the `/wp-content/plugins/webp-express` directory, or install the plugin through the WordPress plugins screen directly.
2. Activate the plugin through the 'Plugins' screen in WordPress
3. Configure it (the plugin doesn't do anything until configured)
4. Verify that it works

You configure the plugin in *Settings > WebP Express*.

WebPExpress uses [WebPConvert](https://github.com/rosell-dk/webp-convert) to convert images. This is how WebPConvert describes itself:

*The current state of WebP conversion in PHP is this: There are several ways to do it, but they all require something of the server setup. What works on one shared host might not work on another. WebPConvert combines these methods by iterating over them (optionally in the desired order) until one of them is successful - or all of them fail.*

These different converting methods are called *converters*.

The best converter is *cwebp*. So the first thing you should do is test whether the cwebp converter is working. Simply click "test" next to the converter. If it doesn't work, you can disable that converter (or try to make it work, by changing the server setup).
The next converter to try is *wpc*, which is equally good as cwebp in terms of quality / filesize ratio, but which is slower. wpc is an open source cloud service, which you will have to install on some other server. If this is too much work, continue to the next converter. In the converter settings, you can read about the individual converters. You can also head to the WebPConvert readme for more information.

Once, you have a converter, that works, when you click the "test"-button, you are ready to test the whole stack, and the rewrite rules. To do this, first make sure to select something other than "Do not convert any images!" in *Image types to convert*. Next, click "Save settings". This will save settings, as well as update the *.htaccess*.

If you are working in a browser that supports webp (ie Google Chrome), you will see a link "Convert test image (show debug)" after a successful save. Click that to test if it works. The screen should show a textual report of the conversion process. If it shows an image, it means that the *.htaccess* redirection isn't working. It may be that your server just needs some time. Some servers has set up caching.

Note that the plugin does not change any HTML. In the HTML the image src is still set to ie "example.jpg". To verify that the plugin is working (without clicking the test button), do the following:

- Open the page in Google Chrome
- Right-click the page and choose "Inspect"
- Click the "Network" tab
- Reload the page
- Find a jpeg or png image in the list. In the "type" column, it should say "webp"

In order to test that the image is not being reconverted every time, look at the Response headers of the image. There should be a "X-WebP-On-Demand" header. It should say "Converting image (handed over to WebPConvertAndServe)" the first time, but "Serving existing converted image" on subsequent requests (WebP-Express is based upon [WebP On Demand](https://github.com/rosell-dk/webp-on-demand)). When routed to image converter, there should also be some headers beginning with "X-WebP-Convert-And-Serve", which reveals information about the conversion.

You can also append `?debug` after any image url, in order to run a conversion, and see the conversion report. Btw: If you append `?reconvert` after an image url, you will force a reconversion of the image.

### Notes

*Note:*
The redirect rules created in *.htaccess* are pointing to the WebP on demand script. If you happen to change the url path of your plugins, the rules will have to be updated. The *.htaccess* also passes the path to wp-content (relative to document root) to the script, so the script knows where to find its configuration and where to store converted images. So again, if you move the wp-content folder, or perhaps moves Wordpress to a subfolder, the rules will have to be updated. As moving these things around is is a rare situation, WebP Express are not using any resources monitoring this. However, it will do the check when you visit the settings page.

*Note:*
Do not simply remove the plugin without deactivating it first. Deactivation takes care of removing the rules in the *.htaccess* file. With the rules there, but converter gone, your Google Chrome visitors will not see any jpeg images.

*Note:*
The plugin has not been tested in multisite configurations. It's on the roadmap!


## Limitations

* The plugin does not work on Microsoft IIS server
* The plugin has not been tested with multisite installation (it is on the roadmap!).
* There might be compatability issues with other plugins. For example *.htaccess* rules from other plugins might interfere.

## Frequently Asked Questions

### How do I set the WebP quality?
You don't. The plugin will try to detect the quality of the source file, and use same quality as that. You do however have an option to select a maximum quality - usefull, because there is seldom any need for a quality above 85 on ordinary web content. In case quality of the source file cannot be determined (that feature requires that Imagick or GraphicsMagick is installed), it will be set to 80.

### How do I make this work with a CDN?
Chances are that the default setting of your CDN is not to forward any headers to your origin server. But the plugin needs the "Accept" header, because this is where the information is whether the browser accepts webp images or not. You will therefore have to make sure to configure your CDN to forward the "Accept" header.

The plugin takes care of setting the "Vary" HTTP header to "Accept" when routing WebP images. When the CDN sees this, it knows that the response varies, depending on the "Accept" header. The CDN is thus instructed not to cache the response on URL only, but also on the "Accept" header. This means that it will store an image for every accept header it meets. Luckily, there are (not that many variants for images)[https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation/List_of_default_Accept_values#Values_for_an_image], so it is not an issue.

### How do I donate?
Putting this question in the "frequently" asked questions section is of course some mixture of humour, sarcasm and wishful thinking. In case there really is someone out there wanting to donate, you can simply write to me, and we can arrange. My contact information is available here https://www.bitwise-it.dk/contact. I have paypal and of course an ordinary bank account.

## Changes in 0.5.0
This version works on many more setups than the previous. Also uses less resources and handles when images are changed.

* Configuration is now stored in a separate configuration file instead of storing directly in the *.htaccess* file and passing it on via query string. When updating, these settings are migrated automatically.
* Handles setups where Wordpress has been given its own directory (both methods mentioned [here](https://codex.wordpress.org/Giving_WordPress_Its_Own_Directory))
* Handles setups where *wp-content* has been moved, even out of Wordpress root.
* Handles setups where Uploads folder has been moved, even out of *wp-content*.
* Handles setups where Plugins folder has been moved, even out of *wp-content* or out of Wordpress root
* Is not as likely to be subject to firewalls blocking requests (in 0.4.0, we passed all options in a querystring, and that could trigger firewalls under some circumstances)
* Is not as likely to be subject to rewrite rules from other plugins interfering. WebP Express now stores the .htaccess in the wp-content folder (if you allow it). As this is deeper than the root folder, the rules in here takes precedence over rules in the main *.htaccess*
* The *.htaccess* now passes the complete absulute path to the source file instead of a relative path. This is a less error-prone method.
* Reconverts the webp, if source image has changed
* Now runs on version 1.0.0 of [WebP On Demand](https://github.com/rosell-dk/webp-on-demand). Previously ran on 0.3.0
* Now takes care of only loading the PHP classes when needed in order not to slow down your Wordpress. The frontend will only need to process four lines of code. The backend footprint is also quite small now (80 lines of code of hooks)
* Now works in Wordpress 4.0 - 4.6.
* Added cache-breaking tokens to image test links
* Denies deactivation if rewrite rules could not be removed
* Refactored thoroughly
* More helpful texts.
* Extensive testing. Tested on Wordpress 4.0, 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7, 4.8 and 4.9. Tested with PHP 5.6, PHP 7.0 and PHP 7.1. Tested on Apache and LiteSpeed. Tested when missing various write permissions. Tested migration. Tested when installed in root, in subfolder, when Wordpress has its own directory (both methods), when wp-content is moved out of Wordpress directory, when plugins is moved out of Wordpress directory, when both of them are moved and when uploads have been moved.

For more info, see the closed issues on the 0.5.0 milestone on our github repository: https://github.com/rosell-dk/webp-express/milestone/2?closed=1

# Roadmap

* Share converter with other sites. Optionally provide conversion service for other sites, which will be able to connect to it using the "wpc" converter.
* Test on multisite
* Display whether the server is able to detect quality of jpegs or not
* Make the fallback quality configurable (the quality to use, when quality of source file cannot be determined)
