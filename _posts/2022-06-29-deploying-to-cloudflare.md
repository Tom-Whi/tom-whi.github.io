---
title: Deploying my blog to Cloudflare
author: TomWhi
date: 2022-06-29
categories: [Website]
tags: [website]
---

I noticed it's possible to deploy a page to Cloudflare.  I thought I should give it a go and deploy this website to Cloudflare to act as another point of access (if for example in the unlikely event my github pages site became unavailable and I was in deperate need to read something here... Ha.) 

I linked cloudflare to my Github account, it asked for permission to a repo, which I have it access to the one this is hosted from. 

![cloudflare-screenshot](/assets/images/2022-06-29%2020_00_46-Clipboard.png){: .shadow }
_Cloudflare Screenshot_

I've then created a webhook inside Cloudflare

`Pages -> Project -> Settings -> Build & Deployments -> Deployment Hooks`

![cloudflare-hook](/assets/images/2022-06-29%2020_10_08-Clipboard.png){: .shadow }
_Cloudflare Hook Screenshot_

Then finally I went into Github and created a webhook under the repo when the page_build action happens. 

![github-hook](/assets/images/2022-06-29%2020_01_09-Webhooks.png){: .shadow }
_Github Hook Screenshot_

It now appears my website is also available on Cloudflare's network

<https://tom-whi-github-io.pages.dev/>

It's totally pointless.  But it's good to know! 