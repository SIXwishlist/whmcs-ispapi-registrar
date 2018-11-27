# WHMCS "ISPAPI" Registrar Module #

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/hexonet/php-sdk/blob/master/CONTRIBUTING.md)
[![Slack Widget](https://camo.githubusercontent.com/984828c0b020357921853f59eaaa65aaee755542/68747470733a2f2f73332e65752d63656e7472616c2d312e616d617a6f6e6177732e636f6d2f6e6774756e612f6a6f696e2d75732d6f6e2d736c61636b2e706e67)](https://hexonet-sdk.slack.com/messages/CD9AVRQ6N)

This Repository covers the WHMCS Registrar Module of HEXONET. It provides the following features in WHMCS:

## Supported Features ##

* Domain registration
* Domain transfer (with AuthInfo code support)
* Domain management
* Domain renewal
* DNS management
* Email forwarding
* URL forwarding
* Optional TLS/SSL for API connection
* Support for Whois Privacy / ID Protection
* Support for all bulk update operations
* Support for IDNs
* Support for new domain sync method (_Sync)
* Support for SRV records
* Support for DNSSEC Management
* Supported testing environment

... and MORE!

## Resources ##

* [Usage Guide](https://github.com/hexonet/whmcs-ispapi-registrar/blob/master/README.md#usage-guide)
* [Release Notes](https://github.com/hexonet/whmcs-ispapi-registrar/releases)
* [Development Guide](https://github.com/hexonet/whmcs-ispapi-registrar/wiki/Development-Guide)

NOTE: We introduced sematic-release starting with v1.0.63. This is why older Release Versions do not appear in the [current changelog](https://github.com/hexonet/whmcs-ispapi-registrar/blob/master/HISTORY.md). But these versions appear in the [release overview](https://github.com/hexonet/whmcs-ispapi-registrar/releases) and in the [old changelog](https://github.com/hexonet/whmcs-ispapi-registrar/blob/master/HISTORY.old).

## Usage Guide ##

Download the ZIP archive including the latest release version [here](https://github.com/hexonet/whmcs-ispapi-registrar/raw/master/whmcs-ispapi-registrar-latest.zip).

## Minimum Requirements ##

For the latest WHMCS minimum system requirements, please refer to
[https://docs.whmcs.com/System_Requirements](https://docs.whmcs.com/System_Requirements)

### Installation ###

Copy all files from the install/ subdirectory to your WHMCS installation root directory ($YOUR_WHMCS_ROOT), while keeping the folder structure intact.

E.g.
`install/modules/registrars/ispapi/ispapi.php => $YOUR_WHMCS_ROOT/modules/registrars/ispapi/ispapi.php`

### Update ###

To update the module, the right way is to simply upload the new version via FTP.

### Configuration ###

#### Reseller Credentials ####

Within the WHMCS Admin area, go to `Setup > Products > Domain Registrars`

Activate the new registrar ”ISPAPI v1.x.x”, and enter your HEXONET credentials. You can create your account [here](https://www.hexonet.net/sign-up)
If you want to use the “TestMode”, you have to create your OTE account [here](https://www.hexonet.net/signup-ote)
pictureTODO

> Make sure that the account you are using is set to renewalmode AUTOEXPIRE in your Control Panel, otherwise domains might be renewed without being paid for by the customer

### Cronjob configuration ###

In order to have your WHMCS always synchronized with our systems, you should make sure to run the WHMCS Domain Sync at least once per day:
`php -q $YOUR_WHMCS_ROOT/crons/domainsync.php`

More information about the Domain Sync can be found [here](http://docs.whmcs.com/Domains_Tab#Domain_Sync_Enabled)

### Nameservers ###

If you want to use DNS, URL or Email forwarding, domains must resolve to the ISPAPI nameserver cluster (ns1.ispapi.net, ns2.ispapi.net, ns3.ispapi.net).

You might want to enter them in `Setup > General Settings > Domains` as Default Nameservers for your customers:
picture TODO
You can also create your own nameserver hostnames and use them, as long as they are registered and resolve to the correct IP addresses.

### Widget activation ###

If you would like to keep your HEXONET WHMCS modules up to date, you should activate the “Hexonet” widget which will display a message with a download link as soon as a new version is available.
To activate it, go  in `Setup > Staff Management > Administrator Roles` select your administrator role and activate the “Hexonet” widget.

picture TODO

### Additional domain fields ###

The registration of some extensions requires sometimes additional domain fields. (e.g. Legal Type and CIRA Agreement for .CA domain)
In order to provide this additional fields on the registration page and map them with our module you have to extend the **$additionaldomainfields** array.

`Prior to WHMCS 7.0, this file was located at /includes/additionaldomainfields.php`
`From WHMCS 7.0 on, to add new additional domains fields, create a new file named additionalfields.php within the /resources/domains/ directory.`

Our module supports the following config fields: **Ispapi-Name, Ispapi-Options, Ispapi-Replacements, Ispapi-IgnoreForCountries** and **Ispapi-Eval**.

    * **Ispapi-Name** (string): Name of the parameter in Ispapi
    * **Ispapi-Options** (string): Comma-separated list of parameter values, the order must match the ones in **Options**. The mapping of values from Options to **Ispapi-Options** is stored in **Ispapi-Replacements**
    * **Ispapi-Replacements** (array): If the parameter value is a valid key of this array, it gets replaced by the respective value. Indirect use via Ispapi-Options is preferred, direct use makes sense to map older values for backwards compatibility.
    * **Ispapi-Eval** (string): PHP code to execute, the parameter value after all replacements is stored in $value

In our module you will find a file named “additionaldomainfields_sample.php” containing a lot of preconfigured extensions running with our systems. (updated on a regular basis) This file will assist you configuring the WHMCS additional domain fields.

Examples:
    `// Maps the "Application Purpose" value on the Ispapi parameter
    // "X-US-NEXUS-APPPURPOSE".
    // e.g. for "Government purposes": X-US-NEXUS-APPPURPOSE=P5
    $additionaldomainfields[".us"][] = array(
                    "Name" => "Application Purpose",
                    "Type" => "dropdown",
                    "Ispapi-Name" => "X-US-NEXUS-APPPURPOSE",
                    "Ispapi-Options" => ",P1,P2,P3,P4,P5",
                    "Options" => ",P1 - Business use for profit,P2 - Non-profit business; club; association; religious organization; etc.,P3 - Personal Use,P4 - Educational purposes,P5 - Government purposes",
                    "Default" => "",
                    "Required" => true`

    `// Maps the "Nexus Country" value on the Ispapi parameter
    // "X-US-NEXUS-VALIDATOR", converting to uppercase.
    // e.g. for "de": X-US-NEXUS-VALIDATOR=DE
    $additionaldomainfields[".us"][] = array(
            "Name" => "Nexus Country",
            "Type" => "text",
            "Description" => "<div>Specify the two-letter country-code of the registrant (if Nexus Category is either C31 or C32).</div>",
            "Size" => "2",
            "Default" => "",
            "Required" => false,
            "Ispapi-Name" => "X-US-NEXUS-VALIDATOR",
            "Ispapi-Eval" => '$value = strtoupper($value);'
    );`

    `// Custom local presence service for .it
    $my_local_presence = array(
            "FIRSTNAME" => "John",
            "LASTNAME" => "Doe",
            "ORGANIZATION" => "Local Presence",
            "STREET" => "Via Presence 123",
            "CITY" => "Milano",
            "STATE" => "MI",
            "ZIP" => "12345",
            "COUNTRY" => "IT",
            "PHONE" => "+39.123456",
            "EMAIL" => 'info@localprecense.it'
    );

    $additionaldomainfields[".it"][] = array(
            "Name" => "Local Presence",
            "Type" => "dropdown",
            "Options" => ",Use Local Presence Service",
            "Default" => "",
            "Required" => false,
            "Ispapi-Options" => "0,1",
            "Ispapi-Eval" => 'if ( $value ) {
                    $command["OWNERCONTACT0"] = $my_local_presence;
                    $command["ADMINCONTACT0"] = $my_local_presence;
            }'
    );`

### HEXONET as Lookup Provider ###

The HEXONET lookup provider yields high performance availability checks of the domains using our fast API. For the domains that are not supported by HEXONET, the lookup fallbacks to WHOIS. In order to utilize this feature, you just have to choose “HEXONET” (The one with the logo HEXONET ISPAPI) in the lookup provider suggestions.
`Setup > Products/Services > Domain Pricing`

picture-TODO

> This new lookup feature requires at least WHMCS version 7.3

Configure the lookup provider by activating the suggestion engine for accurate and efficient search results.

picture-TODO

The Suggestion Engine provides fast domain suggestions based on the searched domain or keyword.
HEXONET is now providing Aftermarket and Registry premium domains support. (Without having to install our ISPAPI High Performance DomainChecker Module)
In order to see premium domains suggestions in the search results, configure the “Premium Domains” section in WHMCS.

picture-TODO

In order to configure your price markups for premium domains you can use the “Configure” button.

picture-TODO

### Search example ###

picture-TODO

### UTF-8 Support ###

For **WHMCS < 5**, if the system charset for WHMCS is set to utf8, the ISPAPI module will enable some workarounds to support UTF-8 for domain registrations, transfers and contact updates.
For **WHMCS >= 5**, it uses the original unmodified parameters as provided by the system.
**WHMCS >= 6** decided to strip the umlauts in domain names and contacts. This module is able to fix this issue and will keep the umlauts.
In any case, the module should be able to handle umlauts and special characters in whois contacts, as long as the respective registry supports it.
For registries that don't support umlauts, some characters will be mapped automatically, e.g. é to e, ö to oe and so on.

### IDN Support ###

In order to support IDN domains it is required to activate ‘Allow IDN Domains’ option in the WHMCS Admin area under:
`Setup > General Settings > Domains > Allow IDN Domains`

### Domain Name WHOIS Privacy ###

The registrar module fully supports the WHOIS Privacy service WHOISTRUSTEE.com:

picture-TODO

Support for WHOIS Privacy (aka ID Protection in WHMCS) been integrated in two ways:
**Support for the new WHMCS 'toggle ID protection' functionality**

Whenever the ID protection flag gets enabled or disabled in the WHMCS Admin area, the module synchronizes the new protection status to the domain.

This functionality is compatible to the WHMCS API “domaintoggleidprotect” call.

>If the ID Protection flag gets disabled in the Admin area, clients will not be able to re-enable WHOIS Privacy themselves, but it can only be re-enabled by an admin!

### Client area WHOIS Privacy management ###

The registrar module registers a new client area function, which will only be visible when using a template thats compatible to the new WHMCS 5 default one.
In the **Management Tools** section of a domain that got ID Protection enabled, there's a new item called **WHOIS Privacy**.

picture-TODO

Using that new function, a client can verify the current protection status, and enable or disable it automatically.
The new WHOIS Privacy page looks as follows (protection is enabled in example):

picture-TODO

### Special features ###

**.CA Registrant WHOIS Privacy**
The new WHOIS Privacy page for .CA domain names looks as follows (protection is enabled in example):

picture-TODO

**.CA Change of Registrant**
The Change of Registrant page for .CA domain names is available under the Management Tools of a .CA domain.

**.IT .CH .SE .SG .LI Change of Registrant**
The change of Registrant of these TLDs requires a special procedure called TRADE. This function is available under the Management Tools of the domain.

**.SWISS Registrations**h
Our registrar module is now supporting .SWISS registrations. (This function requires WHMCS 6.x)
.SWISS domain names are different and cannot be registered with the normal **AddDomain** command.
In order to apply for a .SWISS domain name it is always required to request a domain application with the **AddDomainApplication** command and to provide the class 'GOLIVE'.

Moreover .SWISS registrations require 2 additional domain fields. Please add the following code at the end of the **/resources/domains/additionalfields.php** file:
`## .SWISS DOMAIN REQUIREMENTS ##
$additionaldomainfields[".swiss"] = array();
$additionaldomainfields[".swiss"][] = array(
                "Name" => "Enterprise ID",
                "Type" => "text",
                "Required" => true,
                "Description" => "(must be CHE followed by 9 digits)",
                "Ispapi-Name" => "X-SWISS-REGISTRANT-ENTERPRISE-ID",
);
$additionaldomainfields[".swiss"][] = array(
                "Name" => "Intended use",
                "Type" => "text",
                "Required" => true,
                "Ispapi-Name" => "X-CORE-INTENDED-USE",
);`

This function uses HOOKS and in order to activate these hooks you have to go to:  
`Setup > Products/Services > Domain registrars`, click the “Configure” button in the ISPAPI module line and finally click “Save Changes”.

**Example of a .SWISS registration:**

    * Customer register a .SWISS domain (like he will register a normal domain)
    * Once the invoice paid, the domain application will be sent
    * The domain will be set to PENDING until the application process has been completed
    * A cron job is checking the application status once a day
        - If the application is successful the domain will be set to ACTIVE and the customer will be able to manage it.
        - If the application is failed, the domain will be set to CANCELLED and you will have to refund the customer manually.

In order to work automatically the WHMCS Cron job has to be configured properly. Please have a look at this [page](http://docs.whmcs.com/Crons)

Further information regarding .SWISS is available in our [wiki](https://wiki.hexonet.net/wiki/SWISS)

### SRV Records support ###

WHMCS doesn't allow SRV records. Our module makes it possible !
In order to activate that your have to update the clientareadomaindns.tpl file in your template directory.
In this file you will find 2 &lt;select&gt; tags. In the first you will have to add the following code:

`<option value="SRV"{if $dnsrecord.type eq "SRV"} selected="selected"{/if} >SRV</option>`

In the second you should add:
`<option value="SRV">SRV</option>`

Between the 2 tags  there is an if statement, here you have to replace
`{if $dnsrecord.type eq "MX"}`
with
`{if $dnsrecord.type eq "MX" || $dnsrecord.type eq "SRV"}`

Now you are ready to support SRV records !

### DNSSEC Management support ###

> Please be advised that HEXONET’s nameserver cluster doesn't support DNSSEC yet, so you will need to setup your own DNSSEC enabled nameservers for now.

In order to activate this feature, within the WHMCS Admin area, go to
`Setup > Products/Services > Domain Registrars > ISPAPI`
and activate the **DNSSEC** checkbox.

In the domain details a new “DNSSEC Management” section will be displayed.
This feature allows your customers to add DS and KEY records and set the maxSigLife.

picture - TODO

### Ispapi Domain Import Addon ###

This package includes an domain importer addon. You can use it to automatically import domain names from your HEXONET account into your WHMCS.
To activate it, go in `Setup > Addon Modules` and activate the **Ispapi Domain Import**.
Once done, you will find a new menu item called Ispapi Domain Import under `Addon`

picture - TODO

Just enter the list of domains into the **Domains** field, or pull the list from your account, select a payment gateway, a default currency for new client accounts, and press **Import Domains**.

For each domain, the tool will create a new WHMCS client account using the selected currency and the registrant info, if there's no existing WHMCS client account using the same email address.

Also it will use the current renewal prices as the recurring amounts of imported domain names, using the respective client account currency. 

## Contributing ##

Please read [our development guide](https://github.com/hexonet/whmcs-ispapi-registrar/wiki/Development-Guide) for details on our code of conduct, and the process for submitting pull requests to us.

## Authors ##

* **Anthony Schneider** - *development* - [AnthonySchn](https://github.com/anthonyschn)
* **Kai Schwarz** - *development* - [PapaKai](https://github.com/papakai)
* **Tulasi Seelamkurthi** - *development* - [Tulsi91](https://github.com/tulsi91)

See also the list of [contributors](https://github.com/hexonet/whmcs-ispapi-registrar/graphs/contributors) who participated in this project.

## License ##

This project is licensed under the MIT License - see the [LICENSE](https://github.com/hexonet/whmcs-ispapi-registrar/blob/master/LICENSE) file for details.

[HEXONET GmbH](https://hexonet.net)
