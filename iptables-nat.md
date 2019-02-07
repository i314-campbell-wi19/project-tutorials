# Project 3 resources 


## Resources
* [Iptables and Netfilter Deep Dive](https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture)
* [Linux Network Administrators Guide - Masquerading](http://www.oreilly.com/openbook/linag2/book/ch11.html)
* [DigitalOcean Tutorial - Manipulate Iptables Rules](https://www.digitalocean.com/community/tutorials/how-to-list-and-delete-iptables-firewall-rules)
 

--- 
## Simple Routing Template
You can find a NAT IP-tables template along with instructions on how to fill it out in this repo at `./iptables_nat_template.conf`.

---
## Introduction to IP-tables

### Tables
Iptables uses a basic structure called a table to group rules according to their main function. Common tables include `nat`, `filter`, `mangle`, and `raw`. For this exercise, we'll focus on the **`nat`** table to implement _address translation_ and the **`filter`** table to protect our network from outside traffic. 

### Chains
Within each table, rules are assigned to a structure called a chain. The order of rules in each chain matters. Once a rule is matched, iptables will jump to a specified action (or another chain) without processing the other rules. 

The default chains correspond to different points in the packet lifecycle. Within the `filter` table, we will often work with the `INPUT`, `OUTPUT`, and `FORWARD` chains. These chains correspond to packets sent to the node, packets being sent by the node, and packets being routed through the node respectively. 

Within the `nat` table, our current focus will be on the `POSTROUTING` chain. This chain represents the last set of rules that will be evaluated before forwarding a filtered packet, which is where we want to apply address mappings related to SNAT and masquerading. In contrast, the `PREROUTING` chain is applied before any other process and is used to specify DNAT rules.

### Managing Rules from the Command Line
The `iptables` command allows you to manipulate rules directly from the command line (`sudo` is required). When you create a rule from the command line, specify the table using the `-t` option (filter is the default if you omit the option). 

We can append new rules to the end of a chain using the `-A` option or insert rules at the beginning of a chain using `-I`. Likewise, we can delete a rule from a chain by specifying `-D` and providing a numeric index to the rule.

> Note: These are examples only. Keep reading to learn how to build complete rules.

```
# Append a rule to the POSTROUTING chain of the nat table
iptables -t nat -A POSTROUTING  

# Delete the first rule from the FORWARD chain of filter
iptables -D FORWARD 1           
```

### Customize Rules with Filter Conditions

In addition to specifying table and chain, our rules need to describe the criteria associated with packets we want to filter or manipulate. In this exercise, our _primary consideration_ is the _direction of traffic_ for a given packet. For example, we want to translate the address of packets being forwarded in the **&lt;LAN&gt;** and out the **&lt;WAN&gt;** interface. 

> IMPORTANT: Throughout the instructions, we'll refer to the wired interface as **&lt;LAN&gt;** (because it serves our local network) and the wireless interface as the **&lt;WAN&gt;** (because it connects us to the Internet). Your configuration files will reflect the Raspbian-assigned interface names, such as: **`eth0`** and **`wlan0`**.

We can specify this restriction in our `nat` rule by providing the `-o` option with the name of the corresponding interface. Likewise, we can instruct iptables to look at packets coming from the **&lt;LAN&gt;** and to the **&lt;WAN&gt;** by combining the `-i` option with `-o` in our filter rules.

```
iptables -t NAT -A POSTROUTING -o <WAN>
iptables -A FORWARD -i <LAN> -o <WAN>
```

### Using Connection State in Rules
Iptables is stateful, meaning that it is even possible to specify rules based on the relationship of one packet to others that have been received. This feature shapes our rules in two ways. 

First, you'll see that our masquerading rule also handles the de-masquerading process for incoming packets. 

Second, you'll see that we can base filtering decisions on connection state itself. While it's appropriate for NAT routers to allow outgoing traffic, we typically want to place greater restrictions on incoming traffic. In this assignment, we'll block incoming traffic unless it relates to existing connections from the LAN. To accomplish this feat, we will make use of netfilter's connection tracking extensions which are specified with the `-m conntrack` and `--ctstate RELATED,ESTABLISHED` options

### Specify an Action
The last component of an iptables rule is specified using the `-j` option. In a simple configuration, this option provides the action to be carried out against the packet after the match. In more complex configurations, `-j` can point to a user-defined chain containing additional rules. As you gain experience, you'll see that this provides a greater degree of organization and control flow. While we will encounter the `MASQUERADE` option in our NAT rules, we will more frequently with basic `ACCEPT` and `DROP` rules within the filter context. Bringing this option together with the earlier examples, we see that we can express complete rules using the following syntax:

```
iptables -t nat -A POSTROUTING -o <WAN> -j MASQUERADE
iptables -A FORWARD -i <WAN> -o <LAN> -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

### Default Actions
When you write iptables rules, keep in mind the default actions that will be taken if no other rule matches in a chain. Default actions are configurable through policies that are defined for each chain. From a security point-of-view, it's a good practice to set default policies to block traffic that wasn't explicitly allowed.

```
# Don't forward packets unless there was an explicit rule match
iptables -P FORWARD DROP  
```

Be careful, however, that you don't lock yourself out by a bad combination of rules and policies. For example, setting `-P INPUT DROP` and `-P OUTPUT DROP` will block all inbound and outbound traffic to your Pi if you don't already have rules set up to explicitly allow SSH or other tools.
