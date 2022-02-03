---
title: Launch Essentials
subtitle: Advanced cURLs
description: Part five of our Launch Essentials guide covers using advanced cURL techniques to prepare a site for launch.
anchorid: advanced-curls
layout: guide
showtoc: true
categories: [go-live]
tags: [backup, launch, webops]
type: guide
permalink: docs/guides/launch/advanced-curls/
editpath: launch/05-advanced-curls.md
image: getting-started-Largethumb
---

In this lesson, we’ll use advanced cURL techniques to Test a domain not resolved to Pantheon yet, use the Pantheon Debugger, and suppress certificate errors.

## Test a Domain not Resolved to Pantheon

When you’re migrating a site, you may want to ensure that the site responds correctly to the given host name, but if the DNS is not yet pointed at Pantheon, you won’t be able to do this. A common way around this is to use the `/etc/hosts` file to change your local DNS resolution. Another way is to use cURL’s `--resolve` option, which provides a custom address for a specific host and port pair. cURL `--resolve` requests inserts the specified address into cURL's DNS cache. This overrides the DNS lookup and prevents the normally resolved IP address from being used. 

For example, if you wanted to request `myexamplesite.com` from `23.185.0.1`:

```bash
curl https://www.myexamplesite.com --resolve www.myexamplesite.com:443:23.185.0.1
```

If you want to compare your old site with your new site without making DNS changes:

```bash
diff <(curl -s https://myexamplesite.com) <(curl -s --resolve myexamplesite.com:443:23.185.0.1 https://mycoolwebsite.com)
```

This will show line-by-line differences between the two sites while only resolving the second site from Pantheon’s platform using that domain.

## Use the Pantheon Debugger

You can use the cURL `-I -H` option with Pantheon's Debug header to see additional information about a request.

 ```bash
 curl -I -H “Pantheon-Debug” https://myexamplesite.com
 ```

This command shows the following information:

 ```bash
 HTTP/2 200
 cache-control: public, max-age=3600
 content-language: en
 content-type: text/html; charset=utf-8
 etag: W/"1620823485-0"
 expires: Sun, 19 Nov 1978 05:00:00 GMT
 last-modified: Wed, 12 May 2021 12:44:45 GMT        
 link: <https://pantheon.io/>; rel="canonical"       
 server: nginx
 strict-transport-security: max-age=31622400
 surrogate-key-raw:
 x-drupal-cache: HIT
 x-frame-options: SAMEORIGIN
 x-pantheon-styx-hostname: styx-fe2-b-d65d59d6b-s7n8b
 x-styx-req-id: e415f5fd-b31f-11eb-b9b0-0a6939d335f4 
 date: Wed, 12 May 2021 13:17:50 GMT
 x-served-by: cache-mdw17325-MDW, cache-fty21371-FTY 
 x-cache: HIT, HIT
 x-cache-hits: 1, 1
 x-timer: S1620825471.689334,VS0,VE1
 policy-doc-cache: HIT
 policy-doc-surrogate-key: pantheon.io
 pcontext-pdocclustering: on
 pcontext-enforce-https: full
 pcontext-request-restarts: 1
 vary: Accept-Encoding, Cookie, Cookie, Cookie
 age: 1959
 accept-ranges: bytes
 via: 1.1 varnish, 1.1 varnish
 content-length: 156572
 ```

 ### Understanding Additional Header Information

<dl>

<dt>Cache-control:</dt> 

<dd>This header tells the browser how long it should cache the URL. </dd>

<dt>Etag, Expires, last-modified:</dt> 

<dd>This shows more information about the cache control mechanism, primarily for the browser side.</dd>

<dt>Surrogate-key-raw:</dt>

<dd>This is the cache key that Fastly uses to cache the content if it’s different from the URL path. If you’re using Pantheon’s Advanced Page Cache module/plugin, you’ll see values here, such as the Posts or Nodes that are included in a page. Modifying such values can easily clear the cache for any URLs that include those items.</dd>

<dt>X-served-by:</dt>

<dd> This indicates the Fastly data centers that your request traveled through to reach the `appserver`. </dd>

<dt> X-cache:</dt>

<dd>This generally has the same number of values as `served-by`, and indicates a HIT or MISS for each point. </dd>

<dt>Age:</dt> 

<dd> This shows how long the content has been cached. This generally reflects cache cleared from the Pantheon dashboard, as well as the cache limited by the cache control mechanisms listed above. In some cases this value may exceed your expected max value. This is the result of the `appserver` responding with a `304 not modified` error when Fastly checked for new content.</dd>

<dt>Via:</dt>

<dd>This indicates services in the response chain and the HTTPS protocol (not the version of the service) that was used to respond.</dd>

</dl>

## Suppress Certificate Errors

cURL uses a bundle of Certificate Authority (CA) public keys (CA certificates) to verify the SSL certificate by default.

The `-k` option tells cURL to skip certificate verification, including the server’s TLS certificate and CA certificates. If your SSL certificate is not yet provisioned, using the `-k` option will suppress errors and allow you to get results from your cURL commands.

If you are using SCP and SFTP for transfers, the `-k` option instructs cURL to skip the known hosts verification. 

 ```bash
 curl -k https://myexamplesite.com
 ```