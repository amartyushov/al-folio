---
layout: post
title: Selenium anatomy
date: 2019-07-15
description: 
---
## Simple selenium test example
**INFO.** Selenium version **3.141.59** is used for class diagrams

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


## Selenium client detailed notes
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


### Driver classes
![drivers](http://www.plantuml.com/plantuml/svg/XOvDoi8m443NNqunks-LdY2KGZr1NRWUqx4D9ccO9AA8TrSZb9Q2xlBpvirhOa9EsLdn3pis5sBG5cEa2ACXHjRZJGJKPnm88bdo9Zk9mO1I7Uc4Vh1Krt0NVyOduXDgWZszvzhfzN1Douy37JzBn6ChPN9J8jaNOAilMT0LwAj7ZpovNvwsGBD4h991LJbKYzhv14hc83SLytr9hNSqvFlix1C0)  
Drivers (Chrome, Firefox) are different from each other by different implementations of  
- DriverCommandExecutor
- DriverService
- Capabilities  


### DriverService classes
![DriverService classes](http://www.plantuml.com/plantuml/svg/ZP9DYnD148RFEx_YlQbWui5hM1PM40L1z91pfQTcfqbFrz1LlRj--D-jQJTgOY1UPfYfz-FffcxKg5YTKlSCRqgyv_APotqCZ918bHLZv48bZ5-wcSAUxXZAYNEiqwhm9CQhllcmsNpYO9Jl4bzVurjtrHAEKxkhx0ua7WmodCm0u0Dbhr3OwAeuC2Ztw9biNFI4JEOMy2E7QhHBZATsvUAlTTTItXcYkcXs8EuzZCmV9rh4QKuySRPMrunRtlrYmlRrBUOY9a4IdaV30odUg-HjBFpnCDZn88MXvfXKB7vrFCaldYWovkLdg8dAItY6Zb9J_wSOhznfLH5acYWtqePUTS4MX9737d5ez0Ti1ILdNFzkeRwFIwyxN1YrTRbCwpoB_iy5ECXOhPU7Z55aV-1e1GSMDPd_ubYt8Jjq5kFknx_h4SOYqwNrk6sYuqKuapN4meCyNWXxwxSbfrb7tzXyNot7N9qjFYyz5tlZ1wzJ-Wa0)

### CommandExecutor classes
![CommandExecutor classes](http://www.plantuml.com/plantuml/svg/VL99pvim43t3hvXR5udyWkXJghHIpscbdXtx4LirDl6IT6d-UmSJ3YGGBWZFMvxVi8j9X9GxMx-ZlUywuTINNv0v-K3IYXWHIHi44QJ-NVT2_XGdK8I5Cxbh0ZgVqAXWBWYqtHuWat0dYxSt-bjNnYM4LaWyQVEmEY3staKGoYdUtq4an_U7khhLLKb1NrFg7pIcqUTY_ZHq78misaI-NSDKdoZsa4POymTgoOII5ecknbhAd5JSbRuXN8m4AvIvvRD8GtHb41cUD0cAPcY2Vk0uaeZBZCQx3TE-Q_1d4HdLZw20iWmeojmBGKXD3jU9ngo0SsPGSOew9-MlZaJB9O_eSpPfQezTmB2XXdbJK_RwzhnFgOFlrlSvokmBQANIMUyleSHFLMMd42ARgBdmk9ZMoEa-SIHCT-BqZwlQ_jiCXMmMVvQ5CoOq0w6XvO2pkgW9vgHJy_4bUCtXPH4xE9Kx-ny0)
List of ChromeWebdriver commands:
`https://chromium.googlesource.com/chromium/src/+/master/chrome/test/chromedriver/client/command_executor.py`

### CommandCodec classes
![CommandCodec classes](http://www.plantuml.com/plantuml/svg/bPAnRXin38PdtLDmo-JJaAqmWWJkO6Heqpg0Pvr4-zH8ea7nUYZQldibN-qqXbt4Dnqb_kH7_hhFObY669eRM52C9ha5ERHWWyaPC_GUsR3jVpGxOr_C5Y5ZmEI7E6EC93XpxByZmtp-5QsOj0ruTmS_6-MJ86-CnJU56vMA209k16XpSgKMb4ejoOTpeEuRsX8BGGqtJB7yaFJ8mBXZp9Z4YAVKDWfBHgtUg3qzF7HPs5XPYkmkCrnEW_Af1rTwuPwWT8NrxU8cmngxZzFOesU47PEGfJ0f-b8dVz1wl6s9jrfOZzXDMzed735hC94-Pafds-z1EnAXM53FvwbJnjLVXofNNLy1Vbnz_gINpyzF9y3gIBlrqWzTQodgWtxvHH_CrgD_yRgsNnVzqwjyrTCG77G8hjfXZQ3xnudZX6LlxnZmR9gwpWFrNmDUqPhVpPiNF__LEXfJU_azNBeeqbcLpYP_hfQTdVg-W0un_040)

### Sequence diagram of creating a new session
![Sequence diagram](http://www.plantuml.com/plantuml/svg/bLLXRzem4Fsy_0gh_GBIsBuZCQ52AYfj1ILetQH9o4ckuuPZ8zk1LjF--qoIRl7XLBJVchltthtpivaQoxMjIX7nt2hDSe4WRPIf9tPbG834gXnfA9M5KQ7n0FgANN_6drjx8og8tkqT6dFSSsdP70ngA6PyppCKQ14CuuRbdVzzG5BpFa0shvdzuiBQ_KoL9PT5_0HvPHLfM6bbLQx4mdMOBT-HQHfEoW2oOCvSH1pcMwtAZiCvm3xTFPzxdDMDT5vQIK4VCE-7zeyrcJXVu630CgsE1hItFpQPHCqHiT4dlp9cZOd1CFA-kxuMCVO1Ali5kUqgDlOz3Mzgnp0UIpWEkZp3KtBmm2tuo_603H7TpsaY3GuvXqTU2Tj-TWwaSdJw53FNOzhk8mh1AWFCRg4rsquCzHvv9e3za5Y0jFY8u5Rwp7vF1SeTkvujakMNU3DFxhx6oIR-5czMoN8L9-llVuW1SeQ1Ru6zv05a0HSJ4qQIZyxYptbwzayJzaejP8UW2tYpY6Cb5xnZmyUzP-50KNn4-O8IAD6IX5w4gr_258n19GUN3SIkRszlt5z8eEyWcwp2mAj_k8yVDccSfdVB-uvScED5D5JjlsgDly6m7qPVt3VFwp7GQMcj5QMl-Fz7uTzHDA2Ep3dcHUIiqYUnMAzNRBLCr-oZQNJ2sF1ED9XAsAY0_f72VljZaL3ASEVhcfrYG3wYCQLPwU99gtUrlT-KWa9zocWAigXAyHS0)

### Sequence diagram of calling a regular WebDriver method
![Sequnce diagram](http://www.plantuml.com/plantuml/svg/bPHFJoen4C3FhvzYik_msEGX1syU31697aWmY46zBUq4ghljsau97z-b6xHiPZ6U-vlV_9b9nvuAHw4gnJ-mDHfjDb10S2KCte8dA43QgYgCoaKXoJgGfKP3Octta7aicuXG3HrMbd2edDwXo-3lJ6-sT5C657_gHg-bhWj3i8ZO2jUt4Jnzl2Ug9sskwHvb8Di1sg2poEfoacWcLg5aWGNwsXgF335_ZuQsIpGAcaw5QBk6uNMt1xY23T8WUrhCibnq7daUO-auEEdducww_0zH4oUjTmLXDvZnirD9itojZmKR0pDu9ZufYbiKvsZKp3uEQYEjIimCrLzNkxwFW9uWUvW_Bc7aqNjjpUZvvNLqkDp34FigIjURTBdzjkOYPVeBsIPtr9EDFRScL64SG6h81SVsPpzYu7ueeNj22iykKk4lZk_u6OpHg52Ldm00)


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

   