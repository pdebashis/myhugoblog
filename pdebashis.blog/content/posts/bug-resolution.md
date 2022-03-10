---
title: "Solving a treacherous bug on a Friday night"
date: 2022-03-04T22:34:37+05:30
---

So, I had been trying to run a simple websocket listener. I had a system in place which will do the authentication and provide me with a wss:// address on which I can add a listener. There are a bunch of libraries to do this. So I copy pasted some code from internet which used the ruby gems faye/websocket and EventMachine and all was good for a while. Till I decided to run this thing on a windows machine.  

 

First, we try to setup Ruby in Windows. 

 

Tried installing 3.1 first. All worked till we reached the EventMachine Gem. Here I started getting errors like  

What() : Encryption not enabled on this device. 

 

To resolve this, there were posts suggesting approaches which involved tweaking the Event Machine Gem for windows.

    Build it with ssl option 

    Use gem install -pre for prebuilt gem 

    Use gem install platform ruby 

    Modify a file in the library after installation (add 'require 'em/ruby') 

The last step somewhat worked out and led to a bad file descriptor error which hinted that the library is not suitable for Ruby 3.1 anyway. 

 

Instinctively, I removed 3.1 and installed 2.7 as the original program was running on 2.7 in a linux server anyway. After setting it up and reaching the same Event machine error I had to go through all the above steps again just to reach the same final error message. This time a fanatic googling led me to a github forum which suggested that the Event Machine Gem is not supported on anything later than Ruby 2.4 on Windows.

 

Naturally, I installed Ruby 2.4 and went through the same steps and reached a similar road block. This time it was evident that the functionality which was implemented on Linux cannot be directly ported to Windows. (may be achievable but by someone more capable). Before completely giving up, I decided to use another approach. Years ago, I saw a tech talk on building serverless ruby bots where I heard about Jruby for the first time. 

 

Eventually, I installed Jruby. It’s an interesting approach to run Ruby via JVM and my initial reaction to it was of pure amazement. Slowly, I went through all the steps again to meet at the same dead end of not being able to make the Event Machine thing work. So, It was finally time to change my websocket listener library. Fortunately there is a gem called 'websocket-client-simple' which looked pretty simple and great. But here came the final challenge. 

    Cannot Listen to anything from Code but can get a heartbeat and handshake through IRB 

This issue is a collection of multiple smaller concepts which had to be identified and solved one by one. 

 

Firstly, There is a delay in the websocket.connect function, so if I am checking if connection is established directly after the connect function, I will get an error. This got identified by adding debugging logs and sleep timers throughout the application and fixed with a somewhat adhoc workaround. 

 

Secondly, and the most treacherous challenge was that of the class instance variables not accessible inside the block. This was strange and took a lot of time to figure out. After some searching, I reached a google groups forum post about another library (SunSpot) which had the same issue due to a specific way the method was implemented. Thankfully it had a relatively easy solution to imitate than the explanation which went above my head. The root of the bug is the loss of class scope inside a block in the Websocket object. Solved it by assigning a local variable me = self and accessing "me" inside the block :) 


Finally, once all done I tried testing it in a similar manner as I was using from the beginning, I came to realize that the WebSocket actually was not sending any data at all due to off-hours. Quite satisfied with whatever code I have now but I will be able to test it live on Monday only which is 2 days later.



