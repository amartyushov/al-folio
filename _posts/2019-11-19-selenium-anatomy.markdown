---
layout: post
title: Selenium anatomy
date: 2019-07-15
description: 
---
# Selenium and WebDriver: how does it work
## Simple selenium test example
Below is a simple selenium test, which opens a web page and performs a login operation.  
And after [ChromeDriver](https://chromedriver.chromium.org/downloads) is installed on local machine 
the test will succeed.   
```java
WebDriver driver = new ChromeDriver();
		
driver.get("https://www.saucedemo.com");
driver.findElement(By.id("user-name")).sendKeys("standard_user");
driver.findElement(By.id("password")).sendKeys("secret_sauce");
driver.findElement(By.cssSelector(".btn_action")).submit();
Thread.sleep(10000);

driver.close();
```  
So the goal of the article is to consider each component of the test separately and understand how 
components are communicating to each other.  
## What is WebDriver
Lets start with a general picture  
![general picture](http://www.plantuml.com/plantuml/png/NOzDJiCm48NtSufHjrKla0MgQ1TT8XA91K8eiL-3XUq9ut4HjuUGe4ZiHlRx-TwnMAzMKwGi7hmxlQaad3NSe3lk2_lVavvRHEHG4xiOa8rZ6BJNhnS-tAqQRlZITG_yY8-AOfJ5m8EOIMAv_WM5D4KaP2lyX64fuad5n4caYzdKGkUtaFcqtnEovpc9N9JgUNUlybbMjc6vQUq_iCTGnzT9rBZXCbkTfDHdN_xJR4ewuzQ9n4AA98RbQmnvf90DUkqTDehDDmzV0RYPjxJYtc5q_D7M5By1)
* **WebDriver is a HTTP compliant protocol**, which specification can be found [here](https://w3c.github.io/webdriver/)   
* **ChromeDriver** is an **implementation** of WebDriver protocol
* ChromeDriver communicates with a browser 
through the DevTools remote debugging interface, which is a WebSockets interface described [here](https://chromedevtools.github.io/devtools-protocol/).
    * The Chrome DevTools Protocol allows for tools to instrument, inspect, debug and profile Chromium, Chrome and other Blink-based browsers.   
* Selenium clients communicate with ChromeDriver by sending HTTP requests  


Some code pointers for ChromeDriver implementation:
* The ChromeDriver sources are in the Chromium tree, and can be checked out by following [these](https://www.chromium.org/developers/how-tos/get-the-code) instructions
* Most of the code is under the src/chrome/test/chormedriver directory
* The main function is in [chromedriver_server.cc](https://code.google.com/p/chromium/codesearch#chromium/src/chrome/test/chromedriver/server/chromedriver_server.cc), which sets up an HttpServer that responds to each HTTP request.
* To follow the code for each WebDriver command => start at [http_handler.cc](https://cs.chromium.org/chromium/src/chrome/test/chromedriver/server/http_handler.cc?g=0), which contains a mapping from each WebDriver command to the C++ function that implements it. 
* To familiarize with the code => pick a command and follow what it does starting from [http_handler.cc](https://cs.chromium.org/chromium/src/chrome/test/chromedriver/server/http_handler.cc?g=0).

## Selenium client notes
`org.openqa.selenium.chrome.ChromeDriverService` - Manages the life and death of a ChromeDriver server.


## Demo with Chrome DevTools protocol
* Start Chrome on macOS with opened remote debugging port 
`/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 --no-first-run --no-default-browser-check --user-data-dir=$(mktemp -d -t 'chrome-remote_data_dir')`
    * as a result a browser session is started with opened port for debugging
* Now we need a client which will send commands to the DevTool protocol.
    * In Selenium case this client is a ChromeDriver
    * For our experiment this can be a client with a front-end for convenience
        * so open another chrome window and open a url `http://localhost:9222`
        * it means that we have connected to the remote debugging port of another browser
* The client gives a list of inspectable pages
* After choosing an inspectable page client fetches HTML, JavaScript and CSS files over HTTP
from that page
* Once loaded, Developer Tools establishes a Web Socket connection to its host and starts exchanging JSON messages with it.
* In order to monitor communication over the DevTools protocol:
    * enable DevTools experiments [link](https://stackoverflow.com/questions/15679579/how-do-i-enable-chrome-devtools-experiments/15680012#15680012)
    * click the ⋮ menu icon in the top-right of the DevTools, and select Settings
    * Select Experiments on the left of settings
    * Turn on "Protocol Monitor", then close and reopen DevTools
    * Now click the ⋮ menu icon again, choose More Tools and then select Protocol monitor.
* It is possible to also issue commands
    * Open dev tools on dev tools. [How to](https://stackoverflow.com/questions/12291138/how-do-you-inspect-the-web-inspector-in-chrome/12291163#12291163)
    * Then within the inner DevTools window call different commands in console, e.g.  
    
```javascript
await Main.sendOverProtocol('Page.navigate', {url: "http://google.com"});

await Main.sendOverProtocol('Emulation.setDeviceMetricsOverride', {
  mobile: true,
  width: 412,
  height: 732,
  deviceScaleFactor: 2.625,
});

const data = await Main.sendOverProtocol("Page.captureScreenshot");
```           

   