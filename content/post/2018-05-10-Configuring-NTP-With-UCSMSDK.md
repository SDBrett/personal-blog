---
author: Brett Johnson
categories:
- UCSMSDK
date: "2018-05-10T00:00:00Z"
description: Configuring NTP settings with UCSMSDK
tags:
- Python
- Cisco
- UCS
title: Configuring NTP with UCSMSDK
---

Correct and consistent NTP configuration is vital for the health and monitoring of a datacenter. Many systems default to getting time from underlying components, such as BIOS. This makes ensuring correct configuration from the ground up important for the systems higher in the dependancy chain.

UCS Manager time settings can be quickly and easily configured using UCSMSDK, even at scale.

If you're unfamiliar with UCSMSDK have a look through this [intro post]({% post_url 2018-03-23-Intro-to-UCSMSDK %}) to get started. 

This post assumes that you have already logged into UCS Manager using the sdk.

#### Setting the timezone

Setting the timezone configures the localized timezone. Depending on your organisation, this configuration may or may not be desired.

The timezone value is a string formated as <Country/City>, for example, I would use "Australia/Melbourne". It's important to note that there is no validation against the timezone string, "Australia/Foo" is accepted as valid so check your spelling.

{{< highlight python >}}
    timezone = "Australia/Melbourne"
    mo = handle.query_dn("sys/svc-ext/datetime-svc")
    mo.timezone = timezone
    mo.policy_owner = "local"
    mo.admin_state = "enabled" 
    mo.port = "0"

    handle.set_mo(mo)
    handle.commit()

{{< / highlight >}}

#### Adding NTP Servers

After the timezone has been set, NTP servers will need to be added to UCS Manager. Run the code for each NTP server to be added, or consider using a function.

The configuration setting is not visible in the UCS Manager web UI, however it can be retrieved through programmatic interfaces.

{{< highlight python >}}

def ntp_server_create(handle, name, descr=""):

    from ucsmsdk.mometa.comm.CommNtpProvider import CommNtpProvider

    mo = CommNtpProvider(parent_mo_or_dn="sys/svc-ext/datetime-svc",
        name=name,
        descr=descr)
    handle.add_mo(mo, True)
    handle.commit()
    return mo

# Mock array of NTP servers from config file
ntpSettings = [{"ntpServer":"0.au.pool.ntp.org", "description":"AU NTP"}, {"ntpServer":"103.51.68.133","description":"US NTP"}]

for ntpSetting in ntpSettings:
    ntp_server_create(handle, ntpSetting['ntpServer'],ntpSetting['description'])

{{< / highlight >}}

#### Removing NTP Servers

Removal of an NTP server is similar to adding an NTP server. 

{{< highlight python >}}

ntpServer = "103.51.68.133"
dn = "sys/svc-ext/datetime-svc/ntp-" + ntpServer
mo = handle.query_dn(dn)
if not mo:
    raise ValueError("NTP Server not found")

handle.remove_mo(mo)
handle.commit()

{{< / highlight >}}

#### Validation of configuration

Configurations are a lot like backups, if they haven't been validated they don't exist.

The function has an additional check to ensure that the description matches what's expected. This can be helpful when using configuration files to configure the environment instead of once off scripts.

{{< highlight python >}}

def ntp_server_exists(handle, name, descr=""):

    dn = "sys/svc-ext/datetime-svc/ntp-" + name
    mo = handle.query_dn(dn)
    if mo:
        if descr and mo.descr != descr:
            return False
        return True
    return False

# Mock array of NTP servers from config file
ntpSettings = [{"ntpServer":"0.au.pool.ntp.org", "description":"AU NTP"}, {"ntpServer":"103.51.68.133","description":"US NTP"}]

for ntpSetting in ntpSettings:
    exists = ntp_server_exists(handle, ntpSetting['ntpServer'],ntpSetting['description'])

    if not exists:
        print("NTP server doesn't exists or not correctly configured")
        ntp_server_create(handle, ntpSetting['ntpServer'],ntpSetting['description'])

{{< / highlight >}}
 
#### Summary

Configuration of NTP settings is a small but critical setting for a healthy environment. For UCS Manager, this can be acheived quickly and easily through the Python SDK.