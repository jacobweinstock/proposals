---
id: 0022
title: Customizable OSIE
status: ideation
authors: Jacob Weinstock <jweinstock@equinix.com>
---

## Summary

The Tinkerbell stack, as far as I can tell, does not support custom ipxe scripts/urls, like netboot.xyz.
Only one in memory installation environment (OSIE) is supported at a time.
It does support a different OSIE, as shown with [hook](https://github.com/tinkerbell/hook), but putting this in place happens outside of the tink cli.
This proposal is to add the ability to do the following through modification of tink server data (hardware/template/workflow), ie `tink-cli`:

* support for custom ipxe scripts/urls
* support any in memory installation environment

## Goals and not Goals

Goals

* support for custom ipxe scripts/urls
* support any in memory installation environment
* customizing these changes are only made through tink cli

Non-Goals

## Content

To be able to support providing workers (bare metal machines) custom ipxe scripts/urls or different OSIEs we need to determine at PXE boot time what PXE parameters (kernel, initrd, cmdline) a worker should be given.
Currently, this would call for boots to have this kind of logic.
Boots is doing a very many things (dhcp server, pxe, tftp, http, syslog, rules engine, etc).
This proposal calls for a re-architecture to allow for the "business rules" of whether a machine should be PXE'd or not and with which options to live in a service dedicated only to this function.
Serving DHCP will be isolated to only serve DHCP.
The work to allow a machine to PXE boot will also be isolated into its own service.
See the architecture diagram [here](./architecture.png).

There are a couple nice side effects of this re-architecture.

* we will be able to integrate with existing DHCP servers
  * allows the use of both dynamic and static DHCP addresses
* we can leverage and contribute to some other open source projects ([pixiecore](https://github.com/danderson/netboot/tree/master/pixiecore), [coredhcp](https://github.com/coredhcp/coredhcp)), whose knowledge and expertise in their respective technologies is arguably greater than ours
* we can focus our efforts around 3 core areas that differential the Tinkerbell stack
  * workflow building - tink server
  * installing operating systems - tink-worker/[actions](https://docs.tinkerbell.org/actions/action-architecture/)
  * rules engine to determine if a machine should pxe or not - dewey server
* simpler code bases as they are more focused and singular in purpose
* isolate and manage change in smaller more focused areas

## Trade Offs

* introducing other unfamiliar code bases
* [pixiecore](https://github.com/danderson/netboot/tree/master/pixiecore) is no longer actively being developed (maybe they will donate it to us!)
* all Tinkerbell components/services won't be owned by the Tinkerbell community
* more services add to the operational complexity/overhead to deploy and maintain
* breaking API changes

## Progress

There is demo setup available [here](https://github.com/jacobweinstock/tinkerbell-next).
The POC code for `dewey` is [here](https://github.com/jacobweinstock/dewey).

> Note - the demo/POC setup doesn't do anything more than boot into the operating system installation environments.
> There will need to be more work done to get a full action run.

## System-context-diagram

![proposed re-architecture](./architecture.png#1)

## APIs

There is a need to re-architect the existing hardware, template and workflow APIs.
This is needed in this proposal to allow for specifying custom ipxe scripts/urls and alternate OSIEs.
This relates to existing proposal [0018](https://github.com/tinkerbell/proposals/pull/25).
You can find a yaml mock of the proposed API changes [here](./api-changes.yaml)
