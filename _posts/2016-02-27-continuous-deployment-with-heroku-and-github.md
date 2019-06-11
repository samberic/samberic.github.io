---
id: 139
title: Continuous Deployment With Heroku and Github
date: 2016-02-27T22:00:41+08:00
author: Sam
layout: post
guid: http://samatkinson.com/?p=139
permalink: /continuous-deployment-with-heroku-and-github/
dsq_thread_id:
  - "4616515394"
categories:
  - geek
---
<strong>This post originally appeared on <a href="https://dzone.com/articles/continuous-deployment-with-heroku-and-github" target="_blank">DZone</a>.</strong>

The movement to Continuous Deployment (CD) has been gathering speed and is widely acknowledged as the way to go. Code is checked in, an automated suite is run, and if passed it is automatically deployed into production. A story is not “done” until it is in production, providing value to the end user and CD gives us the smallest mean time to production for our code. To get to this point we have to have a lot of faith in our test suite and the code base, which ensures we will write more robust systems to cope with this way of working.

There still isn’t a huge heap of tooling available to build a continuous deployment pipeline; it tends to be something people have manually crafted using tools such as Puppet, Ansible and Chef. That’s why when I went to put a project up on <a href="https://www.heroku.com/" target="_blank" rel="nofollow">Heroku</a> for the first time in a while I was pleasantly surprised to see it now supports building your code from GitHub and continuous deployment from that repository.

Let's first discuss what <a href="https://www.heroku.com/" target="_blank" rel="nofollow">Heroku</a> brings the table. It’s a great place to deploy your applications and services in a scalable fashion. You can pretty much just drop any application in any language onto Heroku and it’ll spin it up for you, accessible for the world. Your app is scalable on a simple web dashboard too; start out with a single dyno, but increase if you need the capacity. There’s lots of awesome addons you can throw on too such as papertrail for log alerting and HTTPS certificate hosting. The addons vary in price, but to get a simple process up and running is totally free.

This tutorial presumes you’ve signed up for an account at heroku.com and you have an existing Java web project you’d like to set up for CD which is already in Github.
<h2>Step 0: Github Build</h2>
As with any good continuous integration you want to make sure your tests are all run first. Fortunately this is easy to do with Github and TravisCI. You can sign up using your Github account at<a href="http://travis-ci.org/" target="_blank" rel="nofollow">Travis-CI.org</a> which will then allow you to easily create automated builds which are triggered on every check-in.

Once you’ve signed in, you will be given a list of all your repositories. Flick the toggle switch for the project that you’re setting up, in my case WizChat.

<img class="fr-fin fr-dib" src="https://dzone.com/storage/temp/1005696-screen-shot-2016-01-18-at-23927-pm.png" alt="Image title" width="526" />Travis requires a config file to be placed in the root of your project names .travis.yml. Although there’s a variety of options you can choose to setup, for the purpose of this example I went with the simplest options; telling Travis the project is Java and to build using Oracle JDK 8

<img class="fr-fin fr-dib" src="https://dzone.com/storage/temp/1005697-screen-shot-2016-01-18-at-114720-pm.png" alt="Image title" width="527" />
Check this in and your first build will be automatically triggered. If everything is setup correctly in your project, and your tests are passing, Travis will give you a green build. Either way it should send you an email to let you know.

<img class="fr-fin fr-dib" src="https://dzone.com/storage/temp/1005699-screen-shot-2016-01-18-at-25109-pm.png" alt="Image title" width="526" />

For bonus marks, once you’ve completed your first build you will see a badge indicating the build status. Click this and you’ll be given a variety of ways to integrate the build badge into other sites. Convention dictates placing it at the top of the readme for your project. As my readme is in Markdown format I simply copy and paste the Markdown syntax provided:

<img class="fr-fin fr-dib" src="https://dzone.com/storage/temp/1005700-screen-shot-2016-01-19-at-12647-pm.png" alt="Image title" width="748" />
<h2>Setting Up Your Project for Heroku</h2>
As mentioned before, it’s possible to throw pretty much anything at Heroku and have it run. All it requires first is the creation of a Procfile, used by Heroku to know what to run and how.

For web projects it only requires a single line. Again, there’s lot of extended config that you could dig into if you need finer grain control, but the following should be sufficient to get your web project going:
<div class="CodeMirror cm-s-default">
<div class="CodeMirror-scroll">
<div class="CodeMirror-sizer">
<div>
<div class="CodeMirror-lines">
<div>
<div class="CodeMirror-code">
<div>
<div class="CodeMirror-gutter-wrapper"></div>
<pre>web: java <span class="cm-attribute">-jar</span> target/wizchat.jar</pre>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="CodeMirror-gutters"></div>
</div>
</div>
This tells Heroku the command to run, and that the project is a web project so it needs to let us know what the port is which is done via a system variable. You need to ensure that your project runs on whatever port is handed to it by Heroku else your application won’t run. For example, this is done in SparkJava using the following syntax:
<div class="CodeMirror cm-s-default">
<div class="CodeMirror-scroll">
<div class="CodeMirror-sizer">
<div>
<div class="CodeMirror-lines">
<div>
<div class="CodeMirror-code">
<div></div>
<div>
<div class="CodeMirror-gutter-wrapper"></div>
<pre><span class="cm-variable">port</span>(<span class="cm-variable">parseInt</span>(<span class="cm-variable">ofNullable</span>(<span class="cm-variable">System</span>.<span class="cm-variable">getenv</span>(<span class="cm-string">"PORT"</span>)).<span class="cm-variable">orElse</span>(<span class="cm-string">"8080"</span>)));</pre>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="CodeMirror-gutters"></div>
</div>
</div>
For our Procfile to work it also relies on us having a runnable Jar. This can easily be achieved using the Maven assembly plugin, or the Maven shade plugin.
<div class="CodeMirror cm-s-default">
<div class="CodeMirror-scroll">
<div class="CodeMirror-sizer">
<div>
<div class="CodeMirror-lines">
<div>
<div class="CodeMirror-code">
<div>
<div class="CodeMirror-gutter-wrapper">
<pre class="EnlighterJSRAW" data-enlighter-language="null">&lt;plugins&gt;
            &lt;plugin&gt;
                &lt;artifactId&gt;maven-assembly-plugin&lt;/artifactId&gt;
                &lt;configuration&gt;
                    &lt;archive&gt;
                        &lt;manifest&gt;
                            &lt;mainClass&gt;com.samatkinson.Main&lt;/mainClass&gt;
                        &lt;/manifest&gt;
                    &lt;/archive&gt;
                    &lt;descriptorRefs&gt;
                        &lt;descriptorRef&gt;jar-with-dependencies&lt;/descriptorRef&gt;
                    &lt;/descriptorRefs&gt;
                    &lt;finalName&gt;wizchat&lt;/finalName&gt;
                    &lt;appendAssemblyId&gt;false&lt;/appendAssemblyId&gt;
                &lt;/configuration&gt;
                &lt;executions&gt;
                    &lt;execution&gt;
                        &lt;id&gt;make-assembly&lt;/id&gt;
                        &lt;phase&gt;package&lt;/phase&gt;
                        &lt;goals&gt;
                            &lt;goal&gt;single&lt;/goal&gt;
                        &lt;/goals&gt;
                    &lt;/execution&gt;
                &lt;/executions&gt;
            &lt;/plugin&gt;
</pre>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
This code builds a single fat executable jar with all the dependencies built in. You must set the mainClass and finalName and Maven does the rest.

With everything checked in we now head over to Heroku and login to the dashboard. In the top right is an arrow. Click on it and select “Create new app”.
<img class="fr-fin fr-dib" src="https://dzone.com/storage/temp/1005703-screen-shot-2016-01-18-at-25703-pm.png" alt="Image title" width="300" />
Create a name for your app and select in which region you’d like to run it.
<img class="fr-fin fr-dib" src="https://dzone.com/storage/temp/1005709-screen-shot-2016-01-18-at-25725-pm.png" alt="Image title" width="645" />You can then choose the deployment method: select Github and sign into your Github account when prompted. This should integrate and pull al of your repos in. Search for and select the repository you’re integrating and press “Connect”.
<img class="fr-fin fr-dib" src="https://dzone.com/storage/temp/1005712-screen-shot-2016-01-18-at-25832-pm.png" alt="Image title" width="629" />For the final step, select “Wait for CI to pass before deploy” and click Enable Automatic Deploys. You will now have automated deployment of your application to Heroku every check in if the CI passes! Your application will be available at &lt;appname&gt;.herokuapp.com.
<img class="fr-fin fr-dib" src="https://dzone.com/storage/temp/1005714-screen-shot-2016-01-18-at-25927-pm.png" alt="Image title" width="748" />I highly advise setting up the Heroku Toolbelt so that you can then tail the logs of your application to make sure it started correctly.