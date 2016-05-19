---
layout: post
title: Deploying Jekyll to Azure App Service - Part One
categories: [Azure]
tags: [Jekyll, Azure, Ruby]
comments: true

---
One of the things that got me to use **Jekyll**, a static site generator, was that I could easily build my blog on GitHubPages, customize URLs, host source code on GitHub and deploy it to the cloud. I also wanted to build a **Jekyll** blog and host it on Azure Web Apps. The problem encountered, however is that **Jekyll** is based on Ruby and by default, `Azure App Service` does not support it.

Basically, `Azure App Service` supports continuous deployment from source code control and repository tools like GitHub, BitBucket, and Visual Studio Team Services. But it does not have out-of-the-box support for Ruby, a requirement for building **Jekyll** sites. 

Therefore I had to externalize the continuous delivery with certain batch files through *Kudu*. I also added an extension to Azure Web app. By the way, Kudu is the engine behind git deployments in Azure Web Sites.

![JekyllToAzure](/assets/media/JekyllToAzure.png)


Let's start deploying **Jekyll** to `Azure App Service` (Web app). I will assume you already have your **Jekyll** blog running locally and your repository on GitHub. If you don't, follow [Jekyll's getting started guide](http://jekyllrb.com/docs/quickstart/).


#### Step 1 - Setup your deployment settings and scripts 
You will need to create four files on your site root folder. Gemfile might have been created already.

- .deployment

	*.deployment file will be seen by Kudu and will call deploy.cmd*

    {% highlight jekyll %}
    [config]
    command = deploy.cmd
    {% endhighlight %}

- deploy.cmd

	*deploy.cmd will call getruby.cmd and will ensure Ruby is installed. Also, Kudu will sync the files from **Jekyll** under the _site directory*


	{% highlight jekyll %}
    @if "%SCM_TRACE_LEVEL%" NEQ "4" @echo off

    :: Prerequisites

    :: Verify node.js installed
    where node 2>nul >nul
    IF %ERRORLEVEL% NEQ 0 (
      echo Missing node.js executable, please install node.js, if already installed make sure it can be reached from current environment.
      goto error
    )

    :: Setup

    setlocal enabledelayedexpansion

    SET ARTIFACTS=%~dp0%..\artifacts

    IF NOT DEFINED DEPLOYMENT_SOURCE (
      SET DEPLOYMENT_SOURCE=%~dp0%.
    )

    IF NOT DEFINED DEPLOYMENT_TARGET (
      SET DEPLOYMENT_TARGET=%ARTIFACTS%\wwwroot
    )

    IF NOT DEFINED NEXT_MANIFEST_PATH (
      SET NEXT_MANIFEST_PATH=%ARTIFACTS%\manifest

      IF NOT DEFINED PREVIOUS_MANIFEST_PATH (
        SET PREVIOUS_MANIFEST_PATH=%ARTIFACTS%\manifest
      )
    )

    IF NOT DEFINED KUDU_SYNC_CMD (
      :: Install kudu sync
      echo Installing Kudu Sync
      call npm install kudusync -g --silent
      IF !ERRORLEVEL! NEQ 0 goto error

      :: Locally just running "kuduSync" would also work
      SET KUDU_SYNC_CMD=%appdata%\npm\kuduSync.cmd
    )
    ECHO CALLING GET RUBY

    call :ExecuteCmd "getruby.cmd"

    ECHO WE MADE IT

    :: Deployment

    echo Handling Basic Web Site deployment.

    :: 1. KuduSync
    IF /I "%IN_PLACE_DEPLOYMENT%" NEQ "1" (
      call :ExecuteCmd "%KUDU_SYNC_CMD%" -v 50 -f "%DEPLOYMENT_SOURCE%/_site" -t "%DEPLOYMENT_TARGET%" -n "%NEXT_MANIFEST_PATH%" -p "%PREVIOUS_MANIFEST_PATH%" -i ".git;.hg;.deployment;deploy.cmd"
      IF !ERRORLEVEL! NEQ 0 goto error
    )

    
    :: Post deployment stub
    IF DEFINED POST_DEPLOYMENT_ACTION call "%POST_DEPLOYMENT_ACTION%"
    IF !ERRORLEVEL! NEQ 0 goto error

    goto end

    :: Execute command routine that will echo out when error
    :ExecuteCmd
    setlocal
    set _CMD_=%*
    call %_CMD_%
    if "%ERRORLEVEL%" NEQ "0" echo Failed exitCode=%ERRORLEVEL%, command=%_CMD_%
    exit /b %ERRORLEVEL%

    :error
    endlocal
    echo An error has occurred during web site deployment.
    call :exitSetErrorLevel
    call :exitFromFunction 2>nul

    :exitSetErrorLevel
    exit /b 1

    :exitFromFunction
    ()

    :end
    endlocal
    echo Finished successfully.
	{% endhighlight %}

3. getruby.cmd

	*getruby.cmd file will download the latest version of Ruby and will install it on your `App Service` instance. Also, the script will install all the dependencies in your Gemfile, which is below, and will execute **Jekyll** build command via bundler.*

	{% highlight jekyll %}
    @if "%SCM_TRACE_LEVEL%" NEQ "4" @echo off

    REM Put Ruby in Path
    REM You can also use %TEMP% but it is cleared on site restart. Tools is persistent.
    SET PATH=%PATH%;D:\home\site\deployments\tools\r\ruby-2.2.4-x64-mingw32\bin

    REM I am in the repository folder
    pushd D:\home\site\deployments
    if not exist tools md tools
    cd tools
    if not exist r md r
    cd r
    if exist ruby-2.2.4-x64-mingw32 goto end

    echo No Ruby, need to get it!

    REM Get Ruby and Rails
    REM 64bit
    curl -o ruby224.zip -L https://bintray.com/artifact/download/oneclick/rubyinstaller/ruby-2.2.4-x64-mingw32.7z?direct
    REM Azure puts 7zip here!
    echo START Unzipping Ruby
    SetLocal DisableDelayedExpansion & d:\7zip\7za x -xr!*.ri -y ruby224.zip > rubyout
    echo DONE Unzipping Ruby

    REM Get DevKit to build Ruby native gems  
    REM If you don't need DevKit, rem this out.
    curl -o DevKit.zip http://cdn.rubyinstaller.org/archives/devkits/DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe
    echo START Unzipping DevKit
    d:\7zip\7za x -y -oDevKit DevKit.zip > devkitout
    echo DONE Unzipping DevKit

    REM Init DevKit
    ruby DevKit\dk.rb init

    REM Tell DevKit where Ruby is
    echo --- > config.yml
    echo - D:/home/site/deployments/tools/r/ruby-2.2.4-x64-mingw32 >> config.yml

    REM Setup DevKit
    ruby DevKit\dk.rb install

    REM Update Gem223 until someone fixes the Ruby Windows installer https://github.com/oneclick/rubyinstaller/issues/261
    curl -L -o update.gem https://github.com/rubygems/rubygems/releases/download/v2.2.3/rubygems-update-2.2.3.gem
    call gem install --local update.gem
    call update_rubygems --no-ri --no-rdoc > updaterubygemsout
    ECHO What's our new Rubygems version?
    call gem --version
    call gem uninstall rubygems-update -x

    popd

    :end

    REM Need to be in Reposistory
    cd %DEPLOYMENT_SOURCE%
    cd

    call gem install bundler

    ECHO Bundler install (not update!)
    call bundle install

    cd %DEPLOYMENT_SOURCE%
    cd

    ECHO Running Jekyll
    call bundle exec jekyll build

    REM KuduSync is after this!
	{% endhighlight %}

4. Gemfile

	*add the following:*

	{% highlight jekyll %}
	
	[config]
	command = deploy.cmd
	{% endhighlight %}
	


Don't miss part two, **Create your `Azure Web app` and Deploy**. 

The scripts provide were based on [Scott Hanselman](http://www.hanselman.com/blog/RunningTheRubyMiddlemanStaticSiteGeneratorOnMicrosoftAzure.aspx) and 
[Khalid Abuhakmeh](http://rimdev.io/deploying-jekyll-to-windows-azure-app-services/).


- - -
<a class="btn btn-default" href="https://github.com/jjpinto/jjpinto.github.io"><b>GitHub project site</b></a>










