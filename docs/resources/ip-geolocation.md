---
comments: true
hide:
  - toc
---

# IP Geolocation

!!! information

    The information in this article is best-effort, primarily geared towards the AU/NZ region, and is by no means an exhaustive list.

    Accuracy is not guaranteed as some of it is extremely difficult to verify.

IP Geolocation problems in Australia are an ongoing struggle for many service providers, it's a interesting world where Netflix can think you're in New York whilst Disney thinks you're in the middle of Mongolia. Below is a list of geolocation databases that in aggregate form the basis of most third-party geolocation decision making.

Some Content Providers run their own geolocation databases and seed them from various sources.

The information in this article is best-effort, and not an exhaustive list. It's accuracy is not guaranteed as some of this information is extremely difficult to verify

## IP Addresses detected as VPN

Some providers have problems with their CGNAT IP ranges being detected as VPN subnets - most Streaming companies with published contact details will allow you to submit corrections to them in writing advising of which ranges on your network are CGNAT ranges.

Some customers may get IP Addresses detected as a VPN range by utilising services such as [grass.io](https://grass.io) - the dubious nature of services like this should be explained to them and their use discouraged.

## Geolocation Databases

| Name            | Query URL                                                                              | Correction                                                               | Contact                                                |
| --------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------ |
| dbip            | [:material-link:](https://db-ip.com/)                                                  | Lookup[^1]                                                               | Unknown                                                |
| Digital Element | N/A                                                                                    | [:material-link:](https://www.digitalelement.com/contact-us/)            | [:material-mail:](mailto:ip-data@digitalelement.com)   |
| DRMtoday        | N/A                                                                                    | N/A                                                                      | N/A                                                    |
| GeoComply       | N/A                                                                                    | [:material-link:](https://geocomply.my.site.com/Portal/s/contactsupport) | [:material-mail:](mailto:ipintelligence@geocomply.com) |
| InfoSniper      | [:material-link:](https://infosniper.net/)                                             | [:material-link:](https://infosniper.net/geoip-data-correction.php)      | Unknown                                                |
| IP Geolocation  | [:material-link:](https://ipgeolocation.io/)                                           | Email                                                                    | [:material-mail:](mailto:support@ipgeolocation.io)     |
| IP2Location     | [:material-link:](https://www.ip2location.com/demo)                                    | Email                                                                    | [:material-mail:](mailto:support@ip2location.com)      |
| IPHub           | [:material-link:](https://iphub.info/)                                                 | Lookup[^1]                                                               | Unknown                                                |
| IPInfo          | [:material-link:](https://ipinfo.io/)                                                  | [:material-link:](https://ipinfo.io/contact?s=correction)                | Unknown                                                |
| IPinsight       | [:material-link:](https://ipinsight.io/)                                               | Email                                                                    | [:material-mail:](mailto:william@ipinsight.io)         |
| IPIP            | [:material-link:](https://en.ipip.net/ip.html)                                         | Email                                                                    | [:material-mail:](mailto:sarah@ipip.net)               |
| IPligence       | [:material-link:](https://www.ipligence.com/geolocation)                               | [:material-link:](https://www.ipligence.com/contact)                     | Unknown                                                |
| ir.deto         | N/A                                                                                    | N/A                                                                      | Unknown                                                |
| Neustar         | [:material-link:](https://www.home.neustar/resources/tools/ip-geolocation-lookup-tool) | [:material-link:](https://www.home.neustar/support)                      | Unknown                                                |
| Maxmind         | [:material-link:](https://www.maxmind.com/en/geoip-demo)                               | [:material-link:](https://www.maxmind.com/en/geoip-location-correction/) | Unknown                                                |
| OneTrust        | N/A                                                                                    | N/A                                                                      | Unknown                                                |

## Service Matrix

The below table shows which Service utilises which Geolocation database (if possible), as-well as the known contact point that service provider has for troubleshooting of geolocation problems. Any information marked as Unknown just means that the information is Unknown to the author at the time of writing.

| Service                 | Database(s)                | Contact                                                            | Resources                                                                    | Notes                                                                                                                                                                                       |
| ----------------------- | -------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ABC iView[^2]**       | Geocomply[^18]             | [:material-mail:](mailto:manager.ecd@abc.net.au)                   | N/A                                                                          | ABC iView utilises Akamai services, so Akamai may be a good point of contact if communications with ABC are unfruitful                                                                      |
| **Akamai[^3]**          | Geocomply                  | [:material-mail:](mailto:ccare@akamai.com)                         | N/A                                                                          | Any corrections should be emailed to Akamai with [RFC8805](https://www.rfc-editor.org/rfc/rfc8805#section-toc.1-1.2.2.1.2.1.1) compliant information                                        |
| **Amazon (Prime)[^4]**  | Maxmind                    | N/A                                                                | N/A                                                                          | N/A                                                                                                                                                                                         |
| **Cloudflare[^5]**      | Maxmind                    | N/A                                                                | N/A                                                                          | N/A                                                                                                                                                                                         |
| **Disney+[^6]**         | Digital Element            | [:material-mail:](mailto:techops-distribution@disneystreaming.com) | N/A                                                                          | Geolocation issues should be handled via e-mail directly with Disney before resorting to contacting Digital Element                                                                         |
| **Foxtel[^7]**          | GeoComply, ir.deto         | [:material-mail:](mailto:geoip.issues@foxtel.com.au)               | N/A                                                                          | Any corrections should be emailed to Foxtel with [RFC8805](https://www.rfc-editor.org/rfc/rfc8805#section-toc.1-1.2.2.1.2.1.1) compliant information                                        |
| **HBO+[^8]**            | Digital Element            | [:material-mail:](mailto:ctiaengineers@hbo.com)                    | N/A                                                                          | Geolocation issues should be handled via e-mail directly with HBO before resorting to contacting Digital Element                                                                            |
| **Kayo[^9]**            | Foxtel (ir.deto)           | N/A                                                                | [Support](https://help.kayosports.com.au/s/submitenquiry)                    | Kayo is a tenant of Foxtel's DRM platform (hosted and run by ir.deto)                                                                                                                       |
| **Google[^10]**         | Unknown                    | N/A                                                                | [Support](https://support.google.com/websearch/workflow/9308722)             | Manage your Geolocation information via the [Google ISP Portal](https://isp.google.com)                                                                                                     |
| **Netflix[^11]**        | Unknown                    | [:material-mail:](mailto:geosupport@netflix.com)                   | [Support](https://help.netflix.com/en/node/26100)                            | Netflix utilises IRR records to determine your geolocation where possible, see [RFC8805](https://www.rfc-editor.org/rfc/rfc8805) and [RFC9632](https://www.rfc-editor.org/rfc/rfc9632.html) |
| **Optus Sport[^12]**    | IP2Location, Maxmind       | [:material-mail:](mailto:contentserviceoperations@optus.com.au)    | N/A                                                                          | N/A                                                                                                                                                                                         |
| **Paramount Plus[^13]** | OneTrust[^18]              | N/A                                                                | [Support](https://support.paramountplus.com/s/contactsupport?language=en_US) | N/A                                                                                                                                                                                         |
| **Stan[^14]**           | CastLabs(DRMtoday)[^18]    | N/A                                                                | [Support](https://help.stan.com.au/hc/en-us/requests/new)                    | Stan publish absolutely no contact points anywhere; content is sourced from Google GCP, Fastly and DNS routing is done by Route 53 (AWS)                                                    |
| **Tenplay[^15]**        | GeoComply, Digital Element | [:material-mail:](mailto:contactus@networkten.com.au)              | [Support](https://helpdesk.tenplay.com.au/support/tickets/new)               | N/A                                                                                                                                                                                         |
| **7plus[^16]**          | GeoComply, Maxmind         | N/A                                                                | [Support](https://support.7plus.com.au/hc/en-au/requests/new)                | N/A                                                                                                                                                                                         |
| **9Now[^17]**           | GeoComply, Maxmind         | N/A                                                                | [Support](https://help.9now.com.au/hc/en-au/requests/new)                    | N/A                                                                                                                                                                                         |

[^1]: Correction submissions can be made via the Query URL results page.
[^2]: ABC iView don't share any technical contact details, so this is their NOC e-mail.
[^3]: Akamai are known for some odd traffic steering decisions at best, but ultimately traffic should at-least originate from within the same country that your network is located in.
[^4]: Amazon corrections should be submitted via Maxmind.
[^5]: Cloudflare corrections should be submitted via Maxmind.
[^6]: Disney+ corrections should be submitted via Disney.
[^7]: Foxtel VPN detection issues should be submitted to either Foxtel or GeoComply.
[^8]: HBO corrections should be submitted via HBO.
[^9]: Kayo content is delivered via Cloudfront, Fastly and Akamai CDNs.
[^10]: Google utilises IRR records to determine your geolocation where possible, see [RFC8805](https://www.rfc-editor.org/rfc/rfc8805) and [RFC9632](https://www.rfc-editor.org/rfc/rfc9632.html).
[^11]: Netflix issues relating ranges being detected as a VPN should be directed to [cdnetops@netflix.com](mailto:cdnetops@netflix.com).
[^12]: Optus Sport maintain an internal geolocation database in addition to the use of third-party databases.
[^13]: Paramount Plus appears to have no connection to Tenplay even though they are owned by the same company.
[^14]: Stan utilises CastLabs services extensively, it's highly like that their Geoblocking system is the one integrated into DRMToday, a SaaS DRM product offered by CastLabs.
[^15]: Tenplay may share some infrastructure with Paramount Plus in Australia.
[^16]: 7Plus has no paid streaming services.
[^17]: 9Now appears to have no connection to Stan even though they are owned by the same company.
[^18]: Guesstimate / unverified.
