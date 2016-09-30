## Overview

Monitoring network traffic to help with troubleshooting depends on the networking library you use.  If you are using [[OkHttp|Using-OkHttp]] or [[Retrofit|Consuming-APIs-with-Retrofit]], you can take advantage of Facebook's [Stetho](http://facebook.github.io/stetho) project that will allow you to leverage the Chrome network traffic inspector.  You can walk through [[this guide|Debugging with Stetho]] for more details.

If are you are using [[Android Async Http Client|Using-Android-Async-Http-Client]] or any other networking library, you cannot leverage the Stetho project and Chrome project as noted in this [issue](https://github.com/facebook/stetho/issues/116).  However, you can still setup an HTTP proxy that can intercept the network requests. 

### Setting up a proxy

The process requires two parts: one on the PC that will act as the proxy, and the other on the Android device.  It will also walk you through installing a root SSL certificate that can be used by the proxy to inspect SSL network requests.

***On your PC***:

1. Download a 30-day trial of [Charles Proxy](https://www.charlesproxy.com/download/).  There are versions available for Windows, Mac OS, and Linux.

2. You will need to insert a special root certificate in order to inspect SSL traffic.  Run Charles Proxy and navigate to `Help` -> `SSL Proxying` -> `Install Charles Root Certificate on a Mobile Device or Remote Browser`.

     <img src="http://imgur.com/Ac5QR0x.png"/>

     You should see `Configure your device to use Charles as its HTTP proxy on xxxx:x:x:x:xxxx:xxx:xxxx:xxxxxx:8888, then browse to chls.pro/ssl to download and install the certificate.`    Take note of the URL that you should be using (i.e. chls.pro/ssl).

3. SSL sites that must be inspected must be explicitly declared by going to `Proxy` -> `SSL Proxying Settings`.  Type the domain name and port 443 assuming the site is connecting to a standard SSL port.

     <img src="http://imgur.com/YXTqq93.png"/>

4. Make sure to ascertain your IP address of the machine with the proxy installed.  You can go to the `Help` -> `Local IP Address` to find it:

<img src="http://imgur.com/AwbbEwA.png"/>

***On your phone:***

1. Open your Android device and go to [http://chls.pro/ssl](http://chls.pro/ssl).  Download and install the root certificate on the phone.

2. Go to your WiFi settings and modify the existing network:

     <img src="http://imgur.com/DXqpvWl.png"/>

3. Enter the IP address and port 8888.  
     
    <img src="http://imgur.com/AclSz0z.png"/> 

4. Connect to a web site.  **You need to go back to the PC to grant authorization for the device to connect.***

For more details, take a look at this [blog](https://jaanus.com/debugging-http-on-an-android-phone-or-tablet-with-charles-proxy-for-fun-and-profit/) for using Charles Proxy.