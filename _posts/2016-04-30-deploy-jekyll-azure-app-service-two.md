---
layout: post
title: Deploying Jekyll to Azure App Service - Part Two
categories: [Azure]
tags: [Jekyll, Azure, Ruby]
comments: true

---
In the previous article, we learned about running **Jekyll** through `Azure App Services`, the required deployment settings, and forcing continous deployment by installing _Ruby_ from batch files created in Kudu. In part two of this series, we’ll be focusing on 
creating an _Azure Web App_ and deploying the blog site using the automatic scripts running on Kudu.

### Create your Azure Web app 

When the [Web app](https://azure.microsoft.com/en-us/documentation/articles/app-service-web-get-started/) is created, go to your Web App and under Settings, add a new environment variable `SCMCOMMANDIDLE_TIMEOUT=600`. This one will help to avoid timeout when installing **Jekyll** and all dependencies.

Also under Settings, click on Deployment source and select Continuous deployment. You get a list of all supported services: choose GitHub. If you haven't done it yet, have a look at this [post](https://azure.microsoft.com/en-us/documentation/articles/web-sites-publish-source-control/).

Lastly, under Tools, click on Extensions. Install **Jekyll** extension. Extensions add functionality to your `App service`.

### Deploy

Once the files are committed to your repository, *Kudu* should deploy your site. During the first deployment, it might take a bit longer since _Ruby_ needs to be downloaded and installed. After that it seems that happens in the blink of an eye.

By the way, it would be great if _Ruby_ and DevKit would be included in `Azure Web App`.

### Conclusion

While it appears to be a lot of steps, day-to-day authoring is actually quite simple – I just check-in Markdown files to **_GitHub_** and my blog is updated shortly on `Azure App service`.


- - -
<a class="btn btn-default" href="https://github.com/jjpinto/jjpinto.github.io">GitHub project site</a>










