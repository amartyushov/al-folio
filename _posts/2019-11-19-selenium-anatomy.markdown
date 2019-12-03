---
layout: post
title: Selenium anatomy
date: 2019-07-01
description: 
---
* [Simple selenium test example](#simple-selenium-test-example)
* [General overview of components involved in the test](#general-overview-of-components-involved-in-the-test)
* [Selenium client detailed notes](#selenium-client-detailed-notes)
  * [Desired capabilities](#desired-capabilities)
  * [Driver classes](#driver-classes)
  * [DriverService classes](#driverservice-classes)
  * [CommandExecutor classes](#commandexecutor-classes)
  * [CommandCodec classes](#commandcodec-classes)
  * [Sequence diagram of creating a new session](#sequence-diagram-of-creating-a-new-session)
  * [Sequence diagram of calling a regular WebDriver method](#sequence-diagram-of-calling-a-regular-webdriver-method)
* [ChromeDriver detailed notes](#chromedriver-detailed-notes)
  * [Demo with ChromeDriver](#demo-with-chromedriver)
* [Demo with Chrome DevTools protocol](#demo-with-chrome-devtools-protocol)
* [Conclusion](#conclusion)

## Simple selenium test example
**INFO.** Selenium version **3.141.59** is used for class diagrams   
**INFO.** **ChromeDriver** is used in all explanations and examples.

Below is a simple selenium test, which opens a web page and performs a login operation.  
And after [ChromeDriver](https://chromedriver.chromium.org/downloads) is installed on local machine 
the test will succeed (installation steps are not the scope of this article).   
```java
WebDriver driver = new ChromeDriver();
		
driver.get("https://www.saucedemo.com");
driver.findElement(By.id("user-name")).sendKeys("standard_user");
driver.findElement(By.id("password")).sendKeys("secret_sauce");
driver.findElement(By.cssSelector(".btn_action")).submit();

driver.close();
```  
So the goal of the article is to consider each component of the test separately and understand how 
components are communicating to each other.  


## General overview of components involved in the test
Lets start with a general picture  
![general picture](http://www.plantuml.com/plantuml/png/NOzDJiCm48NtSufHjrKla0MgQ1TT8XA91K8eiL-3XUq9ut4HjuUGe4ZiHlRx-TwnMAzMKwGi7hmxlQaad3NSe3lk2_lVavvRHEHG4xiOa8rZ6BJNhnS-tAqQRlZITG_yY8-AOfJ5m8EOIMAv_WM5D4KaP2lyX64fuad5n4caYzdKGkUtaFcqtnEovpc9N9JgUNUlybbMjc6vQUq_iCTGnzT9rBZXCbkTfDHdN_xJR4ewuzQ9n4AA98RbQmnvf90DUkqTDehDDmzV0RYPjxJYtc5q_D7M5By1)
* **WebDriver is a HTTP compliant protocol**, which specification can be found [here](https://w3c.github.io/webdriver/)   
* **ChromeDriver** is an **implementation** of WebDriver protocol
* ChromeDriver communicates with a browser 
through the DevTools remote debugging interface, which is a WebSockets interface described [here](https://chromedevtools.github.io/devtools-protocol/).
    * The Chrome DevTools Protocol allows for tools to instrument, inspect, debug and profile Chromium, Chrome and other Blink-based browsers.   
* Selenium clients communicate with ChromeDriver by sending HTTP requests  

## Selenium client detailed notes
In this section I want to concentrate on implementation details of Selenium libraries (java).  
Consider class hierarchies, relations between the most important parts and entities, and create a sense of which classes reside in which jars.

The usual sequence of steps for UI test is following:  
1. Start e.g. ChromeDriver from code
1. Establish a new session with a ChromeDriver
1. Send WebDriver commands within the session
1. Close the session with a WebDriver 

Below is an overview of class relations of important parts of Selenium lib:
![class overview](http://www.plantuml.com/plantuml/svg/bPBFIlGm5CNNpLFStVSfP9wW30Cgk1FgmdKcFQt1_2d9xOoWy-uspMnQAs8tXTlVEVpkfOV4AlBehFX7JoV4ay6PGaV63I6oyapQgfGemYesmISnAFscUb22XJUZan4kC6GRpPIdkcwWfs1liT_JXwXAlfcX5nplPqnKnZDYfpJeBZYdVFlm3ZroY3bJDKX3y0c4UOh_LuXqUT-8whBeK8Cw6clO8FsXeBWzCWyh2K7JS_rSZ7y5dlFnDCH5h1UI5XtBssytM4ZBDfpz1hGkgpztcTnjMgzd9L2gVzXZ8Kyoclurksx31_Ws_ojV)

Where  
`org.openqa.selenium.remote.service.DriverService` "Manages the life and death of a native executable driver server."  
`org.openqa.selenium.remote.HttpCommandExecutor` is responsible for performing HTTP calls to WebDriver  
`org.openqa.selenium.remote.RemoteWebDriver` provides API to use in tests  

It is useful to specify class hierarchies for above mentioned high level classes in details:
### Desired capabilities
![capabilities](http://www.plantuml.com/plantuml/svg/XP5DJiCm48NNzIcyOvMU8AgAoeO5Ge8JJ9n7Cy9sel4OL7xk3aM2L67KhkTzdv-UjqL9jARehdOqKSUHzU07Xf24uU0c2i-qXo-8o5nJGnFxjdr0KChxYCt6lxiLPuKyKO3_ap2AMuL8fVZhhgXGKEjscr9LwYAiuvrn-dJ_EuL1neIc5tw1BDlzodO_eVj9USosHf16lQIvGM51l-mqB_08OOhyTcpkt6dEjn_hVdpDQtkHKt2EMXlOINjAwwblfZaoZMa_JzYlp1u3CIOx2oo-QelSrnI_0000)     
Capabilities are a key-value properties which describe which features a user requests for the session.  
More details can be found [here](https://github.com/SeleniumHQ/selenium/wiki/DesiredCapabilities)

### Driver classes
![drivers](http://www.plantuml.com/plantuml/svg/XOvDoi8m443NNqunks-LdY2KGZr1NRWUqx4D9ccO9AA8TrSZb9Q2xlBpvirhOa9EsLdn3pis5sBG5cEa2ACXHjRZJGJKPnm88bdo9Zk9mO1I7Uc4Vh1Krt0NVyOduXDgWZszvzhfzN1Douy37JzBn6ChPN9J8jaNOAilMT0LwAj7ZpovNvwsGBD4h991LJbKYzhv14hc83SLytr9hNSqvFlix1C0)  
Drivers (Chrome, Firefox) are different from each other by different implementations of  
- DriverCommandExecutor
- DriverService
- Capabilities  


### DriverService classes
![DriverService classes](http://www.plantuml.com/plantuml/svg/ZP9DYnD148RFEx_YlQbWui5hM1PM40L1z91pfQTcfqbFrz1LlRj--D-jQJTgOY1UPfYfz-FffcxKg5YTKlSCRqgyv_APotqCZ918bHLZv48bZ5-wcSAUxXZAYNEiqwhm9CQhllcmsNpYO9Jl4bzVurjtrHAEKxkhx0ua7WmodCm0u0Dbhr3OwAeuC2Ztw9biNFI4JEOMy2E7QhHBZATsvUAlTTTItXcYkcXs8EuzZCmV9rh4QKuySRPMrunRtlrYmlRrBUOY9a4IdaV30odUg-HjBFpnCDZn88MXvfXKB7vrFCaldYWovkLdg8dAItY6Zb9J_wSOhznfLH5acYWtqePUTS4MX9737d5ez0Ti1ILdNFzkeRwFIwyxN1YrTRbCwpoB_iy5ECXOhPU7Z55aV-1e1GSMDPd_ubYt8Jjq5kFknx_h4SOYqwNrk6sYuqKuapN4meCyNWXxwxSbfrb7tzXyNot7N9qjFYyz5tlZ1wzJ-Wa0)  
As it was said above DriverService "manages the life and death of a native executable driver server."

### CommandExecutor classes
![CommandExecutor classes](http://www.plantuml.com/plantuml/svg/VL99pvim43t3hvXR5udyWkXJghHIpscbdXtx4LirDl6IT6d-UmSJ3YGGBWZFMvxVi8j9X9GxMx-ZlUywuTINNv0v-K3IYXWHIHi44QJ-NVT2_XGdK8I5Cxbh0ZgVqAXWBWYqtHuWat0dYxSt-bjNnYM4LaWyQVEmEY3staKGoYdUtq4an_U7khhLLKb1NrFg7pIcqUTY_ZHq78misaI-NSDKdoZsa4POymTgoOII5ecknbhAd5JSbRuXN8m4AvIvvRD8GtHb41cUD0cAPcY2Vk0uaeZBZCQx3TE-Q_1d4HdLZw20iWmeojmBGKXD3jU9ngo0SsPGSOew9-MlZaJB9O_eSpPfQezTmB2XXdbJK_RwzhnFgOFlrlSvokmBQANIMUyleSHFLMMd42ARgBdmk9ZMoEa-SIHCT-BqZwlQ_jiCXMmMVvQ5CoOq0w6XvO2pkgW9vgHJy_4bUCtXPH4xE9Kx-ny0)
List of ChromeWebdriver commands:
`https://chromium.googlesource.com/chromium/src/+/master/chrome/test/chromedriver/client/command_executor.py`  
`HttpCommandExecutor` as an implementation of `CommandExecutor` takes care of all HTTP requests to the WebDriver.

### CommandCodec classes
![CommandCodec classes](http://www.plantuml.com/plantuml/svg/bPAnRXin38PdtLDmo-JJaAqmWWJkO6Heqpg0Pvr4-zH8ea7nUYZQldibN-qqXbt4Dnqb_kH7_hhFObY669eRM52C9ha5ERHWWyaPC_GUsR3jVpGxOr_C5Y5ZmEI7E6EC93XpxByZmtp-5QsOj0ruTmS_6-MJ86-CnJU56vMA209k16XpSgKMb4ejoOTpeEuRsX8BGGqtJB7yaFJ8mBXZp9Z4YAVKDWfBHgtUg3qzF7HPs5XPYkmkCrnEW_Af1rTwuPwWT8NrxU8cmngxZzFOesU47PEGfJ0f-b8dVz1wl6s9jrfOZzXDMzed735hC94-Pafds-z1EnAXM53FvwbJnjLVXofNNLy1Vbnz_gINpyzF9y3gIBlrqWzTQodgWtxvHH_CrgD_yRgsNnVzqwjyrTCG77G8hjfXZQ3xnudZX6LlxnZmR9gwpWFrNmDUqPhVpPiNF__LEXfJU_azNBeeqbcLpYP_hfQTdVg-W0un_040)  

### Sequence diagram of creating a new session
Below is a sequence diagram which shows communication between classes which are involved in new session creation. Basically this flow happens when following code is executed in the test:    
`WebDriver driver = new ChromeDriver();`
![Sequence diagram](http://www.plantuml.com/plantuml/svg/bLLXRzem4Fsy_0gh_GBIsBuZCQ52AYfj1ILetQH9o4ckuuPZ8zk1LjF--qoIRl7XLBJVchltthtpivaQoxMjIX7nt2hDSe4WRPIf9tPbG834gXnfA9M5KQ7n0FgANN_6drjx8og8tkqT6dFSSsdP70ngA6PyppCKQ14CuuRbdVzzG5BpFa0shvdzuiBQ_KoL9PT5_0HvPHLfM6bbLQx4mdMOBT-HQHfEoW2oOCvSH1pcMwtAZiCvm3xTFPzxdDMDT5vQIK4VCE-7zeyrcJXVu630CgsE1hItFpQPHCqHiT4dlp9cZOd1CFA-kxuMCVO1Ali5kUqgDlOz3Mzgnp0UIpWEkZp3KtBmm2tuo_603H7TpsaY3GuvXqTU2Tj-TWwaSdJw53FNOzhk8mh1AWFCRg4rsquCzHvv9e3za5Y0jFY8u5Rwp7vF1SeTkvujakMNU3DFxhx6oIR-5czMoN8L9-llVuW1SeQ1Ru6zv05a0HSJ4qQIZyxYptbwzayJzaejP8UW2tYpY6Cb5xnZmyUzP-50KNn4-O8IAD6IX5w4gr_258n19GUN3SIkRszlt5z8eEyWcwp2mAj_k8yVDccSfdVB-uvScED5D5JjlsgDly6m7qPVt3VFwp7GQMcj5QMl-Fz7uTzHDA2Ep3dcHUIiqYUnMAzNRBLCr-oZQNJ2sF1ED9XAsAY0_f72VljZaL3ASEVhcfrYG3wYCQLPwU99gtUrlT-KWa9zocWAigXAyHS0)

### Sequence diagram of calling a regular WebDriver method  
Below is a sequence diagram which illustrates general flow for calling WebDriver commands, all code below follows this pattern.  
```java
driver.get("https://www.saucedemo.com");
driver.findElement(By.id("user-name")).sendKeys("standard_user");
driver.findElement(By.id("password")).sendKeys("secret_sauce");
driver.findElement(By.cssSelector(".btn_action")).submit();
```  
Particularly `driver.get("https://www.saucedemo.com");` is shown on sequence diagram (where "https://www.saucedemo.com" is represented as just a "URL")  
![Sequnce diagram](http://www.plantuml.com/plantuml/svg/bPHFJoen4C3FhvzYik_msEGX1syU31697aWmY46zBUq4ghljsau97z-b6xHiPZ6U-vlV_9b9nvuAHw4gnJ-mDHfjDb10S2KCte8dA43QgYgCoaKXoJgGfKP3Octta7aicuXG3HrMbd2edDwXo-3lJ6-sT5C657_gHg-bhWj3i8ZO2jUt4Jnzl2Ug9sskwHvb8Di1sg2poEfoacWcLg5aWGNwsXgF335_ZuQsIpGAcaw5QBk6uNMt1xY23T8WUrhCibnq7daUO-auEEdducww_0zH4oUjTmLXDvZnirD9itojZmKR0pDu9ZufYbiKvsZKp3uEQYEjIimCrLzNkxwFW9uWUvW_Bc7aqNjjpUZvvNLqkDp34FigIjURTBdzjkOYPVeBsIPtr9EDFRScL64SG6h81SVsPpzYu7ueeNj22iykKk4lZk_u6OpHg52Ldm00)

## ChromeDriver detailed notes
So, once again, **ChromeDriver** is an **implementation** of WebDriver protocol. And WebDriver protocol specification can be found [here](https://w3c.github.io/webdriver/).    
Some code pointers for ChromeDriver implementation:
* The ChromeDriver sources are in the Chromium tree, and can be checked out by following [these](https://www.chromium.org/developers/how-tos/get-the-code) instructions
* Most of the code is under the src/chrome/test/chormedriver directory

So ChromeDriver is basically a HttpServer that responds to HTTP requests. The main function is in [chromedriver_server.cc](https://code.google.com/p/chromium/codesearch#chromium/src/chrome/test/chromedriver/server/chromedriver_server.cc)
* the port for the server is hardcoded and is `9151` [link](https://cs.chromium.org/chromium/src/chrome/test/chromedriver/server/chromedriver_server.cc?l=464), but it can be overridden by the [parameter](https://cs.chromium.org/chromium/src/chrome/test/chromedriver/server/chromedriver_server.cc?l=472)
* To follow the code for each WebDriver command => start at [http_handler.cc](https://cs.chromium.org/chromium/src/chrome/test/chromedriver/server/http_handler.cc?g=0), which contains a mapping from each WebDriver command to the C++ function that implements it.  
    * e.g. lets track how url opening happens  
        *  The mapping between WebDriver HTTP path and further call for DevTools protocol can be found below (and a [link](https://cs.chromium.org/chromium/src/chrome/test/chromedriver/server/http_handler.cc?g=0&l=184))
        ```
        CommandMapping(
                  kPost, "session/:sessionId/url",
                  WrapToCommand("Navigate", base::BindRepeating(&ExecuteGet))),
        ```  
        * WebDriver HTTP path for this command is taken from protocol specification [link](https://w3c.github.io/webdriver/#navigate-to)
        ![Webdriver navigate to url]({{ site.baseurl }}/assets/img/for_posts/Navigate_to_webdriver_method.png)
        * WebDriver HTTP path for this command is taken from protocol specification ([link](https://chromedevtools.github.io/devtools-protocol/tot/Page/#method-navigate))  
        ![Page.navigate method]({{ site.baseurl }}/assets/img/for_posts/Page_navigate_devtools_method.png)    
        * WebDriver will log following as a result of nivagation to the url:   
        ```
        ...
            [1574177057.990][INFO]: [1d778fc2fcf507ea962ab77b87d347b9] COMMAND Navigate {
             "url": "https://www.saucedemo.com"
            }
        ...
            [1574177057.994][DEBUG]: DevTools WebSocket Command: Page.navigate (id=13) 525924E9AAE045751E097D69F1E6C1E1 {
              "url": "https://www.saucedemo.com"
            }
            [1574177058.227][DEBUG]: DevTools WebSocket Response: Page.navigate (id=13) 525924E9AAE045751E097D69F1E6C1E1 {
              "frameId": "525924E9AAE045751E097D69F1E6C1E1",
              "loaderId": "5978A67DB58AE2738DCD871588846FAE"
            } 
        ...
        ```
        
### Demo with ChromeDriver
As a small demo lets work with a ChromeDriver but without Selenium libraries, only performing HTTP calls against driver directly. 
The demo will emulate java code:  
```java
WebDriver driver = new ChromeDriver();
driver.get("https://www.saucedemo.com");
driver.findElement(By.id("user-name")).sendKeys("standard_user");
```  
 
* First **start chromedriver** binaries manually (this step is done in Selenium by `org.openqa.selenium.remote.service.DriverCommandExecutor`)    
``` 
➜  ~ chromedriver --whitelisted-ips=
Starting ChromeDriver 78.0.3904.70 (edb9c9f3de0247fd912a77b7f6cae7447f6d3ad5-refs/branch-heads/3904@{#800}) on port 9515
```  
It is necessary to specify `--whitelisted-ips=` parameter due to the recent changes ([link](https://chromium-review.googlesource.com/c/chromium/src/+/952522))  
* The next step is to **create a session** (this step is done in Selenium by `org.openqa.selenium.remote.ProtocolHandshake#createSession()`)  
```
curl -X POST \
  http://127.0.0.1:9515/session \
  -H 'content-type: application/json' \
  -d '{
    "desiredCapabilities": {
        "caps": {
            "nativeEvents": false,
            "browserName": "chrome",
            "version": "",
            "platform": "ANY"
        }
    }
}'
```  

Response is
     
```
{"sessionId":"aca0212be4a400ee65935221a6ea5e3f","status":0,"value":{"acceptInsecureCerts":false,"acceptSslCerts":false,"applicationCacheEnabled":false,"browserConnectionEnabled":false,"browserName":"chrome","chrome":{"chromedriverVersion":"78.0.3904.70 (edb9c9f3de0247fd912a77b7f6cae7447f6d3ad5-refs/branch-heads/3904@{#800})","userDataDir":"/var/folders/32/vl36v3_n0_z80shcn1wcdynw0000gp/T/.com.google.Chrome.xVvLRQ"},"cssSelectorsEnabled":true,"databaseEnabled":false,"goog:chromeOptions":{"debuggerAddress":"localhost:64933"},"handlesAlerts":true,"hasTouchScreen":false,"javascriptEnabled":true,"locationContextEnabled":true,"mobileEmulationEnabled":false,"nativeEvents":true,"networkConnectionEnabled":false,"pageLoadStrategy":"normal","platform":"Mac OS X","proxy":{},"rotatable":false,"setWindowRect":true,"strictFileInteractability":false,"takesHeapSnapshot":true,"takesScreenshot":true,"timeouts":{"implicit":0,"pageLoad":300000,"script":30000},"unexpectedAlertBehaviour":"ignore","version":"78.0.3904.108","webStorageEnabled":true}}  
```    
Where the most important part is a session id **aca0212be4a400ee65935221a6ea5e3f** (which is random of course)  
* As a next step we need to **open** a `https://www.saucedemo.com` **url** within the session    
So the pattern of WebDriver url for this command is `http://localhost:9515/session/:sessionId/url` ([link](https://w3c.github.io/webdriver/#navigate-to))    
So in current case, within a particular session id the command will look like   
```
curl -X POST \
  http://localhost:9515/session/aca0212be4a400ee65935221a6ea5e3f/url \
  -H 'content-type: application/json' \
  -d '{"url":"https://www.saucedemo.com"}
'
```  
* So lets next **find an element** by id `user-name` and insert a value `standard_user` to this field  
The pattern of Webdriver url for finding element is `http://localhost:9515/session/:sessionId/element` ([link](https://w3c.github.io/webdriver/#find-element))  
Within our session the HTTP call will be  
```
curl -X POST \
  http://localhost:9515/session/aca0212be4a400ee65935221a6ea5e3f/element \
  -H 'content-type: application/json' \
  -d '{"using":"id","value":"user-name"}'
```
The important result here is an element id **0.44000226938784204-1**
```
{
    "sessionId": "aca0212be4a400ee65935221a6ea5e3f",
    "status": 0,
    "value": {
        "ELEMENT": "0.44000226938784204-1"
    }
}
```  
And now having the element id we can **send a value to the element**.  
The url pattern is `http://localhost:9515/session/:sessionID/element/:elementID/value` ([link](https://w3c.github.io/webdriver/#element-send-keys))  
```
curl -X POST \
  http://localhost:9515/session/aca0212be4a400ee65935221a6ea5e3f/element/0.44000226938784204-1/value \
  -H 'content-type: application/json' \
  -d '{"value":["standard_user"]}'
```
So as a result of the demo the value `standard_user` was set to the login field. We have just walked through the equivalent of java steps:  
```java
WebDriver driver = new ChromeDriver();
driver.get("https://www.saucedemo.com");
driver.findElement(By.id("user-name")).sendKeys("standard_user");
```  


## Demo with Chrome DevTools protocol
The remaining component in Selenuim communication chain is a DevTools Chrome [protocol](https://chromedevtools.github.io/devtools-protocol/) which
is used by the WebDriver to communicate with a certain browser (in our case ChromeDriver communicates with Chrome browser).  
As a demo lets repeat our steps from previous demos, which can be expressed as a java code  
```java
WebDriver driver = new ChromeDriver();
driver.get("https://www.saucedemo.com");
```  
but in current case we will directly communicate with a browser (we will emulate WebDriver by ourselves).  

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
* (Optional) In order to monitor communication over the DevTools protocol:
    * enable DevTools experiments [link](https://stackoverflow.com/questions/15679579/how-do-i-enable-chrome-devtools-experiments/15680012#15680012)
    * click the ⋮ menu icon in the top-right of the DevTools, and select Settings
    * Select Experiments on the left of settings
    * Turn on "Protocol Monitor", then close and reopen DevTools
    * Now click the ⋮ menu icon again, choose More Tools and then select Protocol monitor.
* It is possible to also issue commands
    * Open dev tools on dev tools. [How to](https://stackoverflow.com/questions/12291138/how-do-you-inspect-the-web-inspector-in-chrome/12291163#12291163)
        * in our case it means we need to open a DevTools for the browser, which is connected to the remotely debugged Chrome
    * Then within the inner DevTools window call different commands in console, e.g.  
    
* So lets call necessary commands for our demo
After we have connected to `http:localhost:9222` to inspectable page, we can call a command to **open a url** 
```javascript
await Main.sendOverProtocol('Page.navigate', {url: "https://www.saucedemo.com"});
```     
<img src="/assets/img/for_posts/DevTools_open_url.png" alt="devtools"
	title="Opening url using dev tools" width="100%" height="100%" />


Couple of examples of other commands
```javascript
await Main.sendOverProtocol('Emulation.setDeviceMetricsOverride', {
  mobile: true,
  width: 412,
  height: 732,
  deviceScaleFactor: 2.625,
});

const data = await Main.sendOverProtocol("Page.captureScreenshot");
```           
## Conclusion  
In this article we went through each component which compose Selenium based UI tests environment. 
We have looked precisely into java implementation details of Selenium library part, 
experimented directly with ChromeDriver and DevTools remote debugger interface of Chrome. 
I hope that after reading this article you have got a solid understanding of this ecosystem and can prudently
make conclusions while writing/debugging Selenium tests. 
   