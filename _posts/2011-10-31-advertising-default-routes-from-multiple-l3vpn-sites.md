---
layout: post
title: Advertising default routes from multiple L3VPN sites
date: 2011-10-31 17:21
author: rbenigno
comments: true
categories: [bgp, cisco, Networking]
---
<span class="Apple-style-span" style="font-size:26px;font-weight:bold;">Overview</span>

The company I work for has several sites connected through L3VPN providers, and multiple sites have Internet connections.  The desired behavior for Internet traffic is as follows:
<ul>
	<li>Sites with a working Internet connection should advertise a default route into the WAN</li>
	<li>The default route should only be advertised to the WAN when the local firewall is advertising a default into OSPF</li>
	<li>Sites without an Internet connection should use the "closest" site for their Internet traffic (closest is determined by provider's IGP)</li>
</ul>
<div>

The configuration I had in place was only advertising a default from one site, resulting in suboptimal routing for Internet traffic in some sites. I spent some time testing different scenarios and my "duh" moment came while watching debug output (ip routing) scroll by.
<pre>*Oct 31 12:09:58.996: RT: updating bgp 0.0.0.0/0 (0x0):
    via 10.10.50.50  
*Oct 31 12:09:58.996: RT: closer admin distance for 0.0.0.0, flushing 1 routes
*Oct 31 12:09:58.996: RT: add 0.0.0.0/0 via 10.10.50.50, bgp metric [20/0]</pre>
</div>
<h2>Simplified Diagram ("broken")</h2>
<a href="{{ site.baseurl }}/assets/multiple-defaults-to-l3mpls5.png"><img class="alignnone size-full wp-image-25" style="border-color:black;border-style:solid;border-width:1px;" title="Multiple-defaults-to-l3mpls" src="{{ site.baseurl }}/assets/multiple-defaults-to-l3mpls5.png" alt="" width="526" height="595" /></a>

<em>Note: I believe the site advertising the default could vary depending on which site first advertised the default into BGP. This has not been tested.</em>
<h1>Solution</h1>
My initial thought was to change the <a href="http://www.cisco.com/en/US/tech/tk365/technologies_tech_note09186a0080094195.shtml" title="http://www.cisco.com/en/US/tech/tk365/technologies_tech_note09186a0080094195.shtml">administrative distance</a> of the default route coming in via BGP.
<pre>!## Sample from R2. Similar config needed on R1.
ip access-list standard Default_Route
 permit 0.0.0.0
!
router bgp 65020
 network 0.0.0.0
 distance 111 10.10.50.50 0.0.0.0 Default_Route</pre>
With this config in place (on R1 &amp; R2) the administrative distance for the default route will be changed from the eBGP default of 20 to 111 - just above OSPF.  This results in the local OSPF default route being used in the IP routing table, instead of the inbound eBGP route.

<pre>R1#sh ip bgp | i 0.0.0.0|32768        
*  0.0.0.0          10.10.50.50                           0 65000 65010 i
*&gt;                  10.11.192.76            1         32768 i</pre>

<strong>Downside?</strong> Honestly, I couldn't come up with anything that might break by making this change.  But I've been trained to think changing administrative distance is bad, so I feel like there must be something I'm missing.

Because of my concerns with changing the administrative distance I started down the path of <a href="http://blog.ioshints.info/2011/09/responsible-generation-of-bgp-default.html" title="http://blog.ioshints.info/2011/09/responsible-generation-of-bgp-default.html">conditional origination of a BGP default route</a>, as described by Ivan Pepelnjak.  As Ivan explains, the conditions match against the IP routing table.  Since the eBGP route wins out the route-map wouldn't be able to match the FW default route I was looking for.    The EEM option seemed overly complex for the environment I work in.

Am I making a mistake by changing the administrative distance?  It's just one little route...
