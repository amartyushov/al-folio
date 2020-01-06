---
layout: post
title: HTTP notes
date: 2019-12-17
description: 
---
**Disclaimer:** current page is not for educational purposes, it has a intention of notes for myself about HTTP aspects.

## Cookies
![Set-cookie](http://www.plantuml.com/plantuml/svg/BOwnJiOm48FtF8KtOFXVe09KYOKHYOuieJ4qehqhTvVszObKLd_MPuyUsQ9jQxg3CswpwaybzA0TbMUQrqe9FDuwjHvYWD5t5MSI3SBgzn83xdMFgfBN1tm8NgVf3GjpeGruxzgNIzlHqpC-JK-dOJRceEFQTAYXw9Qh-7-nEGkguAj5CyBus-ZXAQMHKlhfFm00)  
`Set-Cookie: <name>=<value>[; <Max-Age>=<age>] [; expires=<date>][; domain=<domain_name>] [; path=<some_path>][; secure][; HttpOnly]`

```java
if (max-age AND expiry) {
    use max-age
}
```  
**Default**: cookie is associated with the _location of current document_ (**domain** as well as **path**).    

**Best practicies:**  
- Cookie security: manage cookie scope  

### Secure (Security)
```java
if (HTTPS && cookie_has_Secure_attribute) {
    transmit_cookie()	
} 
if (HTTP && cookie_has_Secure_attribute) {
    do_not_transmit_cookie    
}
```

### HttpOnly (Security)
A cookie can be set and used over HTTP, but also **directly** on the web browser **via JavaScript**.  
```java
if (XSS_breach && CSP_did_not_help) { 
    if (cookie_has_HttpOnly_flag) {
        JS_can_not_access_cookie
        break;
    }

    inject_Malicious_JS()
    access_Cookie_Value()
}
```

# HTTPS
HTTP transforms data in plain text.  
`SSL 3.0 (@Deprecated, Secure Sockets Layer) <-based_on_SSL_3.0- TLS 1.3 (Transport Layer Security)`  
Goals of HTTPS:  
- **Privacy**: encrypting data traffic
- **Integrity**: data received on either side was not altered unknowingly along the way
- **Authentication**: website you are talking to is who they say they are  

## Symmetric encription
The same key is used for encryption and decryption.  
This is what home WIFI uses. There is one key (password), which is set into router and laptop.
```
ENCRYPTION: encryption_algorithm(data, encryption_key) = cipher_text  
```
![symmetric encryption](http://www.plantuml.com/plantuml/svg/XT0n3e9G3CRndLDqKmSkG8nXWDN54n3ub4ReUxP5ysulGMFY8Cxzfx-cfNcZFer3jg5J6aUuSakGLZaw1ydQWI5E-O4CUeTIiKnJT7JKDTxGLd6ROBxB93XemDaBgezBGm_sdkop-8hqgfGl_PnLzS_i3U_pDTXY4CENNDN_vVK3IqWtV-G9)

## Asymmetric encription
Two different keys are used. One to encrypt, second to decrypt. => _Public key Cryptography_.  
![simple overview](http://www.plantuml.com/plantuml/svg/FSun4e8m48NXFgTudPKNe73mA25vm8v9baaMDRTN6CVo_wf_REQhxJcv2-wjvqoh4i0IgcmcMjmgaPXLRTAtExoVZkiDaVyQi2WRj10ltrrH8n9d6x3jKvA01tzQPLaFhlBqnjD7blXT-000)

## Self-signed certificate
Self-signed certificate is a certificate that is **not** signed by certificate authority (CA).
```java
protocol = https
var certificate
if (certificate.NOT_signed_by_CA()) {
    browser_warning = visit_website()
    if (bypass_warning) {
        possible(() -> man-in-the-middle_attack_with(certificate))
    }
} else {
    no_warning = visit_website()
}
```

## Digital signature
```java
signed_data = sign_data(private_key)
result = verify_signature(public_key)

if (result is correct) {
    certain_that_data_came_from_owner_of_associated_private_key
    certain_that_data_was_not_modified_along_the_way
}
```