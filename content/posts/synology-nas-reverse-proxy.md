---
title: "Synology Nas Reverse Proxy"
date: 2025-03-29T18:49:33-04:00
draft: false
toc: false
images:
tags: 
  - synology
  - networking
---

[How to Install Plausible on Your Synology NAS](https://mariushosting.com/how-to-install-plausible-on-your-synology-nas/) and [Synology: How to Add Wildcard Certificate](https://mariushosting.com/synology-how-to-add-wildcard-certificate/) are great guides to setting up a [Plausible](https://github.com/plausible/community-edition) Docker image on your NAS and exposing it to the internet, but they are long. This blog post is a generic and condensed guide to follow in order to reverse proxy a Docker image running on your NAS and exposing it to the internet. To follow this guide, you will need to own a domain name.

## Expose Your NAS via Dynamic DNS

1. In your NAS "Control Panel", click "Security" from the sidebar
2. Click the "Certificate" tab
3. Select the "Add a new certificate" radio selector
4. Select the "Get a certificate from Let's Encrypt" radio selector
5. In the "Domain name" text field input: "\<your-domain\>.synology.me"
6. In the "Subject Alternative Name" text field input: "\*.\<your-domain\>.synology.me"
  - this last step is important in order to have wild card routes

## Create DNS CNAME

1. From your DNS server, for me AWS Route 53, create a CNAME record
  - for example, I own iacut.one and pointed nas.iacut.one at the value iacutone.synology.me
  - this value needs to match the certificate domain from above

## Reverse Proxy a Docker Image

1. In your NAS "Control Panel", click "Login Portal" from the sidebar
2. Click "Advanced" tab
3. Click "Reverse Proxy" button
4. Click "Create" button
  - note, the Destination port needs to match the port defined in the Docker Compose yml

The best part of this is Synology will auto-renew your CA certifications via Let's Encrypt.
