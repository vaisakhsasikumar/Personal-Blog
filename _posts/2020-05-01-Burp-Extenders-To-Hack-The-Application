---
layout: post
title:  "Burp Extenders Demystified"
description: Introduction to my blog
date:   2020-05-01 13:03:36 +0530
categories: burpsuite
---




### HelloWorld

The following Burp Extension does nothing other than print out Hello World to the extension output.


```
from burp import IBurpExtender


class BurpExtender(IBurpExtender):

    #
    # implement IBurpExtender
    #
    
    # The only method of the IBurpExtender interface.
    # This method is invoked when the extension is loaded and registers
    # an instance of the IBurpExtenderCallbacks interface
    # A set of callback methods that can be used by extensions to perform various actions within Burp.
    # https://portswigger.net/burp/extender/api/burp/iburpextendercallbacks.html
    def registerExtenderCallbacks(self, callbacks):
        # Put the callbacks parameter into a class variable so we have class-level scope
        self._callbacks = callbacks
 
	# helper functions for reading HTTP requests, encoding etc. This extension doesn't need them but most will.
        self._helpers = callbacks.getHelpers()
        # Set the name of our extension, which will appear in the Extender tool when loaded
        callbacks.setExtensionName("Hello World POC")

        print "hello world"
        return
```


### HTTPListener

The following Burp Extension builds slightly on the hello world example. 
It now registers as an HTTP Listener, printing a message whenever Burp sends an HTTP message. 


```
from burp import IBurpExtender
from burp import IHttpListener

class BurpExtender(IBurpExtender, IHttpListener):

    #
    # implement IBurpExtender
    #
    
    # The only method of the IBurpExtender interface.
    # This method is invoked when the extension is loaded and registers
    # an instance of the IBurpExtenderCallbacks interface
    # A set of callback methods that can be used by extensions to perform various actions within Burp.
    # https://portswigger.net/burp/extender/api/burp/iburpextendercallbacks.html
    def registerExtenderCallbacks(self, callbacks):
        # Put the callbacks parameter into a class variable so we have class-level scope
        self._callbacks = callbacks
 		# save the helpers for later. This extension doesn't need them but most will.
        self._helpers = callbacks.getHelpers()
        # Set the name of our extension, which will appear in the Extender tool when loaded
        callbacks.setExtensionName("Hello HTTP POC")

 	# register ourselves as an HTTP listener
 	# The listener will be notified of requests and responses made by any Burp tool. 
 	# Extensions can perform custom analysis or modification of these messages by registering this HTTP listener.
        callbacks.registerHttpListener(self)
        print "hello world"
        return

    def processHttpMessage(self, toolFlag, messageIsRequest, currentMessage):
    	# Invoked when an HTTP request is about to be issued, and when an HTTP response has been received
    	# https://portswigger.net/burp/extender/api/burp/IHttpListener.html
        print "Burp sent an HTTP Request!"
```


### AddHeader
This extension builds on hellohttp.py by actually doing something with the HTTP Message. In doing so it also demonstrates other commonly used syntax. 

In this case:
If an HTTP message goes through burp
...if the message is a request
...if the burp suite tool that sent it was the proxy
...then add a header "Client-IP: 111.222.222.111" to the request

```

from burp import IBurpExtender
from burp import IHttpListener

class BurpExtender(IBurpExtender, IHttpListener):

    #
    # implement IBurpExtender
    #
    
    # The only method of the IBurpExtender interface.
    # This method is invoked when the extension is loaded and registers
    # an instance of the IBurpExtenderCallbacks interface
    # A set of callback methods that can be used by extensions to perform various actions within Burp.
    # https://portswigger.net/burp/extender/api/burp/iburpextendercallbacks.html
    def registerExtenderCallbacks(self, callbacks):
        # Put the callbacks parameter into a class variable so we have class-level scope
        self._callbacks = callbacks
 		# save the helpers for later. This extension doesn't need them but most will.
        self._helpers = callbacks.getHelpers()
        # Set the name of our extension, which will appear in the Extender tool when loaded
        callbacks.setExtensionName("Add HTTP Header")

 	# register ourselves as an HTTP listener
 	# The listener will be notified of requests and responses made by any Burp tool. 
 	# Extensions can perform custom analysis or modification of these messages by registering this HTTP listener.
        callbacks.registerHttpListener(self)
        print "hello world"
        return

    def processHttpMessage(self, toolFlag, messageIsRequest, currentMessage):
    	# processHttpMessage is invoked when an HTTP request is about to be issued, and when an HTTP response has been received
    	# https://portswigger.net/burp/extender/api/burp/IHttpListener.html

        # only operate on Proxy requests, not TOOL_SCANNER, etc etc.
        if toolFlag == self._callbacks.TOOL_PROXY:
        # only operate on requests (not responses)
            if messageIsRequest:
                self.processRequest(currentMessage)
                
    def processRequest(self, currentMessage):
        # boilerplate for getting the HTTP request and splitting it into header and body.

        request = currentMessage.getRequest()
        parsedRequest = self._helpers.analyzeRequest(request)
        requestHeaders = parsedRequest.getHeaders()
        requestBody = self._helpers.bytesToString(request[parsedRequest.getBodyOffset():])

        # This is the actual injected header.
        requestHeaders.add('Client-IP: 111.222.222.111') 

        # build the HTTP request back out of the new request Headers and Body
        updatedRequest = self._helpers.buildHttpMessage(requestHeaders, requestBody)

        # set this newly built request as the current request, replacing the original. 
        currentMessage.setRequest(updatedRequest)
	

```
