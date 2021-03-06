---
layout: post
title: "Setting up a load balancer with failover support in Azure"
date: 2020-12-09
description: "Use Terraform to setup a load balancer with failover support in Azure"
img: azure-loadbalancer.jpg
---

# The cloud solves all your problems..
Lately I was in need of a load balancer solution supporting the classic active-passive scenario. In other words, forward (ideally TCP) traffic to server A. If server A is not reachable reroute to server B. Being a fairly simple and common problem it actually took me quite some time to find a solution I was happy with.

Initially I was very confident that the Azure Load Balancer was exactly what I was looking for. Finding out that it does only support Azure internal endpoints made my anticipation decline. Next stop: Azure Application Gateway!

Being already sceptical cause its kind of an overkill service for such a simple problem I sat down and created an instance anyway. Seeing external targets being an option when configuring the backend pools was good. HTTP and HTTPS being the only supported protocols wasn’t great. Finding out that App Gateway is not capable of my desired failover scenario took it off if my option list.

There I was, sitting in front of cloud resources having terabytes of memory but not being able to realize a simple failover use case. Just before coming to the conclusion of setting up and managing a few HA proxy VMs by myself I stumbled upon a service I have never heard of… Azure Traffic Manager.

After reading about it for a few minutes I was pretty sure that this is what a was looking for. Let’s see if I was right.

# ..but does it, _really_?
Azure Traffic Manager’s functionality is based on DNS. It acts as an DNS alias that is being rewritten depending on the load balancing algorithm, of which it supports the following:

- Priority - provide multiple endpoints, the one that is available and has the highest priority receives all of the traffic
- Weighted - allows to set a weight for each endpoint and distributing the traffic based on it
- Performance - routes the request to the fastest endpoint based on where the request originates from
- Geographic - routes the request to the closest endpoint based on where the request originates from
- Multivalue - returns all available endpoints
- Subnet - allows to map the requesting source IP subnet to a fixed endpoint

For showcasing the general functionality we’ll setup a failover functionality for two DNS servers.

![Failover](/assets/img/failover.png)

The Terraform code for setting up the corresponding Traffic Manager profile can be found below.

{% highlight python %}
resource "azurerm_traffic_manager_profile" "my-traffic-manager-profile" {
  name                   = "my-traffic-manager-profile"
  resource_group_name    = "my-traffic-manager"
  traffic_routing_method = "Priority"

  dns_config {
    relative_name = "my-traffic-manager-profile" # the host name for the FQDN of this traffic manager profile
    ttl           = 60 # the TTL for the DNS alias
  }

  monitor_config { # the config for monitoring the endpoints for availability and latency
    protocol                     = "tcp" # the protocol to monitor with
    port                         = 53 # the port to monitor
    interval_in_seconds          = 30 # the time between probes
    timeout_in_seconds           = 10 # the time the probing agent waits for a response before considering a check as failed
    tolerated_number_of_failures = 3 # the number of failing health probes that will be tolerated before marking the endpoint as unhealthy
  }
}
{% endhighlight %}

For our failover use case the `Priority` method is the one we are going with. We’ll provide two endpoints, a primary and a secondary and the former will have a higher priority (=a lower value). In terms of Terraform code this configuration will look like this.

{% highlight python %}
resource "azurerm_traffic_manager_endpoint" "primary" {
  name                = "cloudflare"
  resource_group_name = "my-traffic-manager"
  profile_name        = "my-traffic-manager-profile"
  target              = "1.1.1.1"
  type                = "externalEndpoints"
  priority            = 1
}

resource "azurerm_traffic_manager_endpoint" "secondary" {
  name                = "google"
  resource_group_name = "my-traffic-manager"
  profile_name        = "my-traffic-manager-profile"
  target              = "8.8.8.8"
  type                = "externalEndpoints"
  priority            = 2
}
{% endhighlight %}

To test our configuration we simply need to constantly send DNS requests to our Traffic Manager instance and meanwhile shutdown the Cloudflare DNS server. 
Afterwards our DNS requests should be rerouted the Google’s DNS server.

To not interrupt DNS queries all over the world I decided to take the less interfering path of just disabling the cloudflare endpoint like so:

```
$ az network traffic-manager endpoint update -g my-traffic-manager --profile-name my-traffic-manager-profile -n cloudflare --endpoint-status Disabled
```

This simulates an outage of our primary endpoint and after a short amount of time the following happens.

```
$ while 1; do host rabbagliettiandrea.github.io my-traffic-manager-profile.trafficmanager.net; sleep 1; done
...
Using domain server:
Name: my-traffic-manager-profile.trafficmanager.net
Address: 1.1.1.1#53
Aliases:

rabbagliettiandrea.github.io has address 185.199.109.153
rabbagliettiandrea.github.io has address 185.199.110.153
rabbagliettiandrea.github.io has address 185.199.108.153
rabbagliettiandrea.github.io has address 185.199.111.153

Using domain server:
Name: my-traffic-manager-profile.trafficmanager.net
Address: 8.8.8.8#53 # notice that the IP changed
Aliases:

rabbagliettiandrea.github.io has address 185.199.109.153
rabbagliettiandrea.github.io has address 185.199.111.153
rabbagliettiandrea.github.io has address 185.199.110.153
rabbagliettiandrea.github.io has address 185.199.108.153
...
```

As you can see the failover to Google’s DNS server works just as expected. And if we re-enable the cloudflare endpoint, requests will be routed back to it again automatically.

This behavior can be accomplished for any kind of protocol running over TCP. Regarding HTTP and HTTPS, Traffic Manager even supports checking a specific path and matching given HTTP status codes to determine whether the endpoint is healthy.

And there you have it. A very easy to configure load balancer with failover support that supports targets within and outside of the Azure cloud.