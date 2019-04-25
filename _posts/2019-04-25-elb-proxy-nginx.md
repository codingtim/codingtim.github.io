---
title: "Forward requests from nginx to an AWS Load Balancer."
tags:
  - devops
---

AWS Load Balancers are given a default DNS name. 
When resolving this DNS name one or more IP addresses will be included in the response.
However these IP addresses will change over time. 
This becomes a problem when you want to use nginx to forward or proxy traffic to the AWS Load Balancer.

Creating a proxy_pass in nginx config is done by creating a location such as: 

    location /my-resource/ {
     proxy_pass http://my-loadbalancer-1234567890.eu-west-1.elb.amazonaws.com/;
    }

This will work just fine and requests to /my-resource/ will be forwarded to the AWS Load Balancer.
After some time this solution will stop working resulting in errors when trying to access the resource.
The reason for this is that nginx will only do the DNS lookup of my-loadbalancer-1234567890.eu-west-1.elb.amazonaws.com once at startup time and cache this until nginx is reloaded or restarted.
So when AWS decides to change the IP addresses of your load balancer the default DNS record is updated but nginx keeps using the cached IP addresses.

AWS will change these IP addresses for various reasons. 
For example when the load increases more IP addresses might be added to handle the increase in traffic. 
Another reason could be that hardware is being replaced inside AWS.

The solution is to get nginx to respect the DNS TTL values. 
This can be done by using variables inside the proxy_pass definition.
Variables are handled differently by nginx than static configuration.

Changing the proxy_pass to use a variable can be done with the following config: 

    resolver 127.0.0.1;
    set $myloadbalancer my-loadbalancer-1234567890.eu-west-1.elb.amazonaws.com$request_uri;
    location /my-resource/ {
     proxy_pass http://$myloadbalancer;
    }

With this configuration nginx will lookup the DNS name and respect the TTL in the response.
Note that we also added a resolver as nginx needs this config for the lookup.
