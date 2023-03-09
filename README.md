# Integration with Shopify

- [Overview](https://paper.dropbox.com/doc/Integration-with-Shopify--AWwjp1YFVnLvktQXWfmutsFeAg-UWEQBPHaJyOQIkNC9Y2bb#:uid=385601843392647891925075&h2=Overview)
- [Installation / Configuration](https://paper.dropbox.com/doc/Integration-with-Shopify--AWwjp1YFVnLvktQXWfmutsFeAg-UWEQBPHaJyOQIkNC9Y2bb#:uid=011940856419557773821248&h2=Installation-/-Configuration)
    - [Shopify Theme Modification](https://paper.dropbox.com/doc/Integration-with-Shopify--AWwjp1YFVnLvktQXWfmutsFeAg-UWEQBPHaJyOQIkNC9Y2bb#:h2=Shopify-Theme-Modification)
    - [settings_schema.json](https://paper.dropbox.com/doc/Integration-with-Shopify--AWwjp1YFVnLvktQXWfmutsFeAg-UWEQBPHaJyOQIkNC9Y2bb#:h2=settings_schema.json)
    - [imgix.liquid](https://paper.dropbox.com/doc/Integration-with-Shopify--AWwjp1YFVnLvktQXWfmutsFeAg-UWEQBPHaJyOQIkNC9Y2bb#:h2=imgix.liquid)
    - [Enabling imgix Integration](https://paper.dropbox.com/doc/Integration-with-Shopify--AWwjp1YFVnLvktQXWfmutsFeAg-UWEQBPHaJyOQIkNC9Y2bb#:h2=Enabling-the-imgix-Integration)
- [Caveats & Warnings](https://paper.dropbox.com/doc/Integration-with-Shopify-AWwjp1YFVnLvktQXWfmutsFeAg-UWEQBPHaJyOQIkNC9Y2bb#:h2=Caveats-&-Warnings)

----------

## Disclaimer: 
## This guide presents a working solution for integrating imgix into your Shopify site. All configurations are made to work on top of Shopify’s front end and CDN, but will have certain limitations that are detailed in our [caveats & warnings](https://paper.dropbox.com/doc/Integration-with-Shopify--AXEJ3MQzJw1tBd~~gICO1WX1Ag-UWEQBPHaJyOQIkNC9Y2bb#:h2=Caveats-&-Warnings) section below. We cannot, unfortunately, guarantee expected results if at any time Shopify were to make changes to its system in ways that create breaking issues with this guide.

## Overview

imgix is easy to integrate with your Shopify store. To get started, you'll need to sign up for imgix and [set up a Source](https://docs.imgix.com/setup/creating-sources).

Once you've created your imgix account, it's time to set up the Source you'll use to serve your Shopify images. Head over to the [Sources section](https://dashboard.imgix.com/sources) of the Dashboard, and click the **Add a Source** button. On the Source creation page, fill out the form with the following information:


- **Subdomain**: Anything you like. In this tutorial, the example will use the subdomain `imgix-shopify`.
- **Source Type**: Web Folder
- **Base URL**: https://cdn.shopify.com

After everything is filled out, click **Create Source** to queue your new Source. On the next page, you should see the **Status** indicator change from **Queued** to **Deployed** after several minutes. At this point, imgix is fully configured and ready to integrate with Shopify!


----------


## Installation / Configuration

**Shopify Theme Modification**

To begin serving your content with imgix, you'll need to make two changes to your Shopify theme. Special thanks to Jason Bowman at *Freakdesign* for creating the [initial version](https://freakdesign.com.au/blogs/news/91099207-how-to-use-imgix-with-shopify) of these files. You can access Shopify's theme file editor by going to `Online store > Themes > Customize > Theme actions > Edit Code`.


**settings_schema.json**

The first change lets you configure your imgix installation from inside Shopify's theme settings. Copy the code below to the top of your theme's `Config/settings_schema.json` file, inside the outermost pair of square brackets (`[ ]`). Make sure to add a comma (`,`) to the file following the `}` preceding where you paste this code in the snippet. You can also search for `settings_schema.json` in the search bar at the top to find it.

```json
{
  "name": "imgix",
  "settings": [
    {
      "type": "paragraph",
      "content": "Check out imgix's \[Shopify integration guide\](https:\/\/docs.imgix.com\/guides\/shopify-integration) to learn more about these options."
    },
    {
      "type": "checkbox",
      "id": "enableImgix",
      "label": "Enable imgix"
    },
    {
      "type": "text",
      "id": "imgixUrl",
      "label": "imgix subdomain",
      "info": "The subdomain you set within imgix. Example: `https:\/\/mysite-shopify.imgix.net`."
    },
    {
      "type": "text",
      "id": "imgixShopifyUrl",
      "label": "Shopify CDN domain",
      "default": "\/\/cdn.shopify.com",
      "info": "Don't change this unless you have a proxy in place. Not sure? Leave it as is."
    }
  ]
},
```

**imgix.liquid**

The second file adds a new Liquid tag that helps users generate imgix URLs. This time, create a new file in the `Snippets` directory of your theme named `imgix.liquid`. Copy the code below into that file, and save it:

```liquid
{% capture IMGIX %}
  {% comment %}
    <!--
    Convert a Shopify CDN path into an imgix path, with parameters.

    * Refer to the imgix documentation to learn about all the available parameters: https://docs.imgix.com/apis/url
    -->
  {% endcomment %}
  {% if settings.enableImgix %}
    {% for i in (1..1) %}
      {% comment %}
        <!-- Check to make sure the src exists, and that imgix url theme settings is not blank. If blank, stop! -->
      {% endcomment %}
      {% unless src or settings.imgixUrl != blank %}
        {{ src }}
        {% break %}
      {% endunless %}

      {% comment %}
        <!-- Check to make sure the src has the Shopify CDN url in it. If it doesn't this does not need to run any further -->
      {% endcomment %}
      {% assign cdnUrl = settings.imgixShopifyUrl | strip %}
      {% unless src contains cdnUrl %}
        {{ src }}
        {% break %}
      {% endunless %}

      {% assign imgixUrl = settings.imgixUrl | strip %}

      {% comment %}
        <!-- Create a list of all the imgix filters we want to use -->
      {% endcomment %}
      {% assign filters = 'bri,con,cs,exp,gam,high,hue,invert,sat,shad,sharp,usm,usmrad,vib,auto,bg,blend,bm,ba,balph,bp,bw,bh,bf,bc,bs,bx,by,border,pad,faceindex,facepad,faces,ch,chromasub,colorquant,dl,dpi,fm,lossless,q,mask,nr,nrs,colors,palette,prefix,dpr,flip,or,rot,crop,fit,h,rect,w,blur,htn,mono,px,sepia,txt,txtalign,txtclip,txtclr,txtfit,txtfont,txtline,txtlineclr,txtpad,txtshad,txtsize,txtwidth,trim,trimcolor,trimmd,trimpad,trimsd,trimtol,mark,markalign,markalpha,markbase,markfit,markh,markpad,markscale,markw,markx,marky' | split:',' %}

      {% comment %}
        <!-- Build up the list of filters to add to the url -->
      {% endcomment %}
      {% assign imgWithQuerystring = "?" %}

      {% if src contains '?' %}
        {% assign imgWithQuerystring = '&' %}
      {% endif %}

      {% for _filter in filters %}
        {% if [_filter] %}
          {% assign imgWithQuerystring = imgWithQuerystring | append:_filter | append:'=' | append:[_filter] | append:'&' %}
        {% endif %}
      {% endfor %}
          
      {% assign modifySrc = src | split:'?' | first | append: "?" %}
      {% assign newSrc = modifySrc | strip | replace:cdnUrl,imgixUrl | append:imgWithQuerystring %}
    {% endfor %}

    {{ newSrc | default:src }}
  {% else %}
    {{ src }}
  {% endif %}
{% endcapture %}{{ IMGIX | strip | replace:'  ' | strip_newlines }}
```

**Enabling the imgix Integration**

Now that you've set up your Shopify theme to work with imgix, you can enable imgix in your theme settings. Head to `Online store > Themes > Customize`. Once there, you'll see an **imgix** option in the left-hand sidebar. Click on this to configure your imgix setup. It should look something like the following:


- **Enable imgix**: Make sure this is checked.
- **imgix** **Subdomain**: Whatever you set your Source subdomain to in step 1. In this example: `https://imgix-shopify.imgix.net`.
- **Shopify CDN** **Domain**: You'll most likely not want to change this from the default value of `//cdn.shopify.com`.

After you've set everything up, click **Save**, and you can move on!

At this point, you're ready to change your existing theme's images to use imgix. The exact process will vary depending on your theme, but the basics will be the same. Just find places in your theme where existing images are being output, and replace them with the imgix Liquid tag you just created. Here's an example:

Before:

`{{ product.featured_image | img_url:'grande' }}`

After:

```liquid
{% assign feat_img_url = product.featured_image | img_url:'master' %}
{% include 'imgix' src:feat_img_url w:600 auto:'format' flip:'v' %}
```

It's a good idea to always use the `master` variant of Shopify's images, and let imgix handle the resizing. That way, you'll always get the best quality image possible. Jason Bowman has created a [great demo page](https://freakdesign.com.au/pages/imgix-and-shopify-optimisation-demo) showcasing more potential uses of this tag.

Here's another example of using imgix to easily build a responsive image using `srcset` and `sizes`: 

```html
{% assign feat_img_url = product.featured_image | img_url:'master' %}

<img
  src="{% include 'imgix' src:feat_img_url w:960 auto:'format' %}"
  srcset="
    {% include 'imgix' src:feat_img_url w:320 auto:'format' %} 320w,
    {% include 'imgix' src:feat_img_url w:640 auto:'format' %} 640w,
    {% include 'imgix' src:feat_img_url w:960 auto:'format' %} 960w,
    {% include 'imgix' src:feat_img_url w:1280 auto:'format' %} 1280w,
    {% include 'imgix' src:feat_img_url w:1600 auto:'format' %} 1600w
  "
  sizes="(min-width: 375px) 50vw, 100vw"
  alt="My awesome product"
>
```

This example will result in an image sized to fill the whole viewport's width when below 375 pixels, and 50% of the viewport's width when at or above 375 pixels. [Eric Portis' post](https://ericportis.com/posts/2014/srcset-sizes/) on `srcset` and `sizes` is a great place to learn more about these responsive image tools.


## Caveats & Warnings

**Native Resizing:**
Shopify’s in-house templating language also allows for image resizing, similar to imgix. However, we generally recommend only resizing your images by passing in imgix parameters, rather than through a combination of both imgix and Shopify.

```html
<ul class="grid product-single__thumbnails" id="ProductThumbs">
  {% for image in product.images %}
    <li class="grid__item">
      <a data-image-id="{{ image.id }}" href="{{ image.src | img_url: '1024x1024' }}" class="product-single__thumbnail">
        <img src="{% include 'imgix' src:image.master h:263 %}" alt="{{ image.alt | escape }}">
      </a>
    </li>
  {% endfor %}
</ul>
```

**Native Image Transformations:**
Shopify also provides its own set of Photo Editor tools for editing product images. We recommend not using this feature in combination with imgix images as the changes will not always take effect, even with image purging.

An illustration:


![Found under the All products tab](https://assets.imgix.net/libraries/integration-guides/shopify/01-an_illustration.png?pad=40&w=926&mask-bg=2B3944&mask=corners&corner-radius=8&bg=2B3944&dpr=2&auto=compress,format)

![Opening an editing an image in Shopify](https://assets.imgix.net/libraries/integration-guides/shopify/02a-all_products.png?mark-y=0&mark-w=620&mark-pad=0&w=1300&pad-right=660&mark-x=960&mark64=aHR0cHM6Ly9hc3NldHMuaW1naXgubmV0L2xpYnJhcmllcy9pbnRlZ3JhdGlvbi1ndWlkZXMvc2hvcGlmeS8wMmItYWxsX3Byb2R1Y3RzLnBuZz9tYXNrPWNvcm5lcnMmY29ybmVyLXJhZGl1cz04&mask-bg=2B3944&mask=corners&corner-radius=8&bg=2B3944&auto=compress,format&pad=20&dpr=1.5)


After saving the image and checking the product page, we see that the changes have not taken effect.

![Saved image does not show image edits from Shopify](https://assets.imgix.net/libraries/integration-guides/shopify/03-green_beans.png?pad=40&w=926&mask-bg=2B3944&mask=corners&corner-radius=8&bg=2B3944&dpr=2&auto=compress,format)
