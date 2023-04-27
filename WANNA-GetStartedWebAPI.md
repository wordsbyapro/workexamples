# Get started with WANNA SDK

<!-- TOC -->
- [Requirements and limitations](#requirements-and-limitations)
- [Try out our demo sample](#try-out-our-demo-sample)
- [Create your own web app](#create-your-own-web-app)
- [How to launch WANNA SDK in WeChat](#how-to-launch-wanna-sdk-in-wechat)
- [Frequently asked questions](#frequently-asked-questions)
<!-- /TOC -->

WANNA SDK enhances your website with virtual try-on experience for footwear and watches, which works directly in a mobile web browser without installing any apps.

## Requirements and limitations

Supported environments: 
* iOS: 
	* Safari 14 and later
	* Google Chrome on iOS 14.3 and later
	* in-app browsers for most common messengers: WhatsApp, Facebook, Telegram, Snapchat, and Instagram
* Android: 
	* Google Chrome 88 and later
	* in-app browsers for most common messengers: WhatsApp, Facebook, Telegram, and Instagram

The trial version has some limitations:

* Only several specific models are supported. You'll have to buy the full version to work with your own 3D assets.
* The 3D model download can't be interrupted. If you request another model before the previous model finished downloading, the new model download will start, but the previous download will complete anyway.

## Try out our demo sample

1. Unzip the sample and go to the **samples** folder.
2. Open the **index.html** file and insert the license string that authorizes your use of WANNA SDK:<br>
    `// Your license string should be here:
    const license = '<PUT_YOUR_LICENSE_STRING_HERE>'
    `
3. Start a HTTP server.
  * PHP:<br>
    `php -S 0.0.0.0:8005`
  * Python 2.x:<br>
    `python -m SimpleHTTPServer 8005`
  * Python 3.x:<br>
    `python -m http.server 8005`

4. Open `http://localhost:8005` in browser.
5. Click the **TryOn** button and wait for initialization. Once the library initializes, the try-on session will begin, using the default model hardcoded into the demo sample.<br>
*Note:* you'll need to allow access to the camera.

## Create your own web app

Host **iframe.html** and WANNA SDK scripts: **core.js** and **sdk.js**. We'll assume that they are located in a **libs** folder alongside your scripts and HTMLs.

*Note:* Some platform providers will restrict the use of the hosted frame. Check the headers for the following settings:

* `frame-ancestors` in the CSP header set to `deny`, or to a host or scheme different from the page that runs WANNA SDK (see https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)
* `X-Frame-options` set to `DENY` (see https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)

These settings may prevent your using the hosted frame on other pages. In that case host our files elsewhere and specify the full paths to these files in **index.html** and the `iframeSrc` parameter of the `init` method.

Create a HTML document that will contain the frame: let's call it **index.html**. **example.js** is where you write your own code. Add the scripts at the end of the `<body>` tag:

    <script defer src="./libs/sdk.js"></script>
    <script defer src="./example.js"></script>

Create a root element inside the `<body>` tag:

    <div id="root"></div>

You may want to hide this element to start with.

Create **example.js**. Put in the basic steps for working with the library:

1. (optional) Check if the current environment supports WANNA SDK. It depends on the device and browser. You may want to hide the virtual try-on button in unsupported environments.<br>
```javascript
await wanna.checkEnvironment()
```

2. Initialize the WANNA library using the `init` method. Pass in the element where the virtual try-on will be rendered, license string that authorizes your use of WANNA SDK, type of session (`'sneakers'` or `'watch'`), the link to the hosted iframe, and optionally the iframe size (by default it takes up the whole container):
```javascript
await wanna.init({
    container: 'root',
    iframeSrc: './libs/iframe.html',
    type: 'sneakers',
    license: '<PUT_YOUR_LICENSE_STRING_HERE>',
})
```
The good time to initialize is when the user goes to the page that offers virtual try-on and you expect them to use it. Note that initialization will take up some processing resources and download traffic on the user's device. To prevent overuse of these resources, avoid initializing too early. On pages that are showing the whole range of content your website offers, wait until the user actually chooses virtual try-on.

3. (optional) Call the `initVideo` method to check and request camera permissions, and start showing the video stream once they are granted. Call it at the same time as the `init` method or immediately afterwards.

The purpose of this step is to show the user some activity while the library is initializing. Once the video is initialized, you may display the `root` element that will have the raw camera output:
```javascript
wanna.initVideo().then(() => {
    showElement('root')
})
```

4. (optional) Download from the CDN the model that will be used for virtual try-on. This will reduce the time to start up the try-on experience, as the model will be loaded and ready when you go to the next step.<br>
```javascript
await wanna.downloadModel({ id: 'wanna01' })
```

If you know the model ID—for example, because of starting up on the particular product's page—we recommend downloading the model immediately after the initialization completes, to save the user's time. If step 2, `init`, has completed before step 3, `initVideo`, just start downloading the model without waiting for the video.

5. Start the try-on session. If you haven't done it beforehand, the method will first download the model, then render it on the screen.<br>
```javascript
await wanna.downloadAndSetModel({ id: 'wanna01' })
```

**Important:** 

- If the user hasn't granted access to the camera, virtual try-on will not work. Handle the `NotAllowedError` to tell the user that camera access is required.
- For non-localhost addresses, HTTPS is required for correct operation of the camera.
- The SDK initialization can fail on unsupported devices. In this case, the `init` method will return an error. Handle errors using `try...catch` statement. Please use the code samples provided together with the SDK as a reference implementation.

The contents of the files may look like this:

### index.html 
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta http-equiv="x-ua-compatible" content="ie=edge">
  <title>WANNABY / WEB-AR</title>
  <meta name="description" content="WANNA WEB-AR">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="mobile-web-app-capable" content="yes">
</head>
<body>
  <button id='start' class='hidden'>Start</button>
  <div id='root'></div>

  <style>
    .hidden {
      display: none;
    }
  </style>
  <script defer src="./libs/sdk.js"></script>
  <script defer src="./example.js"></script>
</body>
</html>
```

### example.js
```javascript
const startBtn = document.getElementById('start')

const handleNotAvailable = () => {
  startBtn.textContent = 'Try-on not available'
  startBtn.disabled = true
}

async function runExample () {
  const canInit = await wanna.checkEnvironment()

  if (canInit) {
    try {
      await Promise.all([
        wanna.initVideo(),
        wanna.init({
         container: 'root',
         iframeSrc: './libs/iframe.html',
         type: 'sneakers',
         license: '<PUT_YOUR_LICENSE_STRING_HERE>',
      }),
      ])
    } catch {
      handleNotAvailable()
      return
    }
    startBtn.classList.remove('hidden')

    startBtn.addEventListener('click', async function () {
      try {
        await wanna.downloadAndSetModel({ id: 'wanna01' })
      } catch (error) {
        if (error.name === 'NotAllowedError') {
	  // ask user to allow camera access or check site settings
	}
        handleNotAvailable()
      }
    })
  }
}

runExample()
```
## How to launch WANNA SDK in WeChat

WeChat Web View can run WANNA SDK. To embed virtual try-on into your WeChat mini-program:

1. Add WANNA SDK to a page on your website as described [above](#create-your-own-web-app).
2. Add a Web View to your WeChat mini-program and set it up to call this webpage. Open the **index.wxml** file (may be called differently in your project) and modify its contents to look like this:

```xml
<!--index.wxml-->
<view class="container">
    <web-view class="web-view" src="<ENTER_THE_WEBPAGE_ADDRESS_HERE>"></web-view>
</view>
```

Now this Web View will run the same virtual try-on experience that the user could obtain by going to your website.

## Frequently asked questions

- [Why isn't virtual try-on displayed even after the model is loaded?](#why-isnt-virtual-try-on-displayed-even-after-the-model-is-loaded)

#### Why isn't virtual try-on displayed even after the model is loaded?

There can be several reasons. Please try the following solutions:

* If you use a loading screen, it may overlay the virtual try-on. Check that you hide the loading screen from view once the virtual try-on starts.
* If you hide the virtual try-on element during loading, make sure that you remove the hidden attribute when virtual try-on is ready.

To ensure that your code knows when the try-on is ready and the model is loaded, use the `await` operator with the WANNA `init` or `downloadAndSetModel` methods. 
