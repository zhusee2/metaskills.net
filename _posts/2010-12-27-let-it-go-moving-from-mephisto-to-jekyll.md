---
layout: post
title: Let It Go - Moving From Mephisto To Jekyll
categories: 
  - ruby-rails
---

<p>
  Welcome to the new MetaSkills.net! I have moved from Mephisto to <a href="https://github.com/mojombo/jekyll">Jekyll</a> and done a complete rewrite of the presentation layer to the latest web standards as a way of staying somewhat sharp with the latest HTML5/CSS3 hotness. In this article, I would like to cover all the gory details of what I learned about moving to Jekyll and the various techniques I have used for the site's CSS. But first a small history lesson. 
</p>

<p>
  If you are somewhat new to rails, Mephisto was one of the greatest open source blog engines a <a href="/2008/03/22/metaskills-net-reborn-on-mephisto/">ruby developer could work with</a>. But ruby, and especially rails, is a fast paced landscape, making it hard to keep any CMS project up to date. Options like file storage, comment moderation, templating languages, hosting locations and more are all very hard to embody in one project. I think that is why Mephisto's creator, <a href="http://twitter.com/#!/technoweenie">Rick Olson</a> (@technoweenie), <a href="http://techno-weenie.net/2010/6/23/you-can-let-go-now/">announced that he was dropping the project</a>. Good for him and many thanks for his hard work on all the open source projects he has contributed to! So let's talk about Jekyll et al.
</p>


<h2>Migrating Mephisto Data</h2>

<p>
  So maybe you have an old Mephisto install that you would like to migrate to Jekyll? If so, I have <a href="https://gist.github.com/756111">posted a gist</a> of some rails 3 models that I used to export my data from MySQL to static files following the format required by Jekyll. Simply populate a new rails 3 project with these, fire up a console and do <code>Site.export_jekyll</code>. This will create a folder in <code>tmp/jekyll_posts</code> and write a file for each Mephisto article. The posts will contain the YAML front matter for jekyll and hopefully a good starting point of the categories you had assigned to that post. As a bonus, the export will create a <code>tmp/jekyll_posts/rewrites.txt</code> file that will be very useful to include in a .htaccess file of your jekyll project. An example below.
</p>

```apache
RewriteRule ^2010/8/19/how-to-clean-a-campfire-room-of-uploads$ /2010/08/19/how-to-clean-a-campfire-room-of-uploads/ [L]
```

<p>
  Also included in the gist is a <code>Site.export_jskit</code> method that will generate an XML file of  your comments to import into Disqus. Though I was able to import my Mephisto comments into Disqus and see them linked from the admin interface, I have yet to get them to show up on the new site. I have a support request open with Disqus and will update this article and the gist with any new information.
</p>


<h2>Using Jekyll</h2>

<p>
  The best way you can learn how to use Jekyll is to <a href="https://github.com/mojombo/jekyll/wiki/Sites">review code of the sites</a> that use it. If you want to learn more about how I have implemented jekyll, you could <a href="https://github.com/metaskills/metaskills.net">browse my source</a> on github.
</p>

<p>
  Like many others, I have created a tasks directory in my project root with some executable scripts that help me author, publish and deploy my site. Here are some examples.
</p>

```bash
#!/usr/bin/env zsh
# File: tasks/scss
# Handy to watch for scss file changes while designing.
sass --scss --watch _sass:_site/resource --style compact --no-cache

#!/usr/bin/env zsh
# File: tasks/jekyll
# Create the site and tidy it.
setopt rmstarsilent
rm -rf _site/* && \
  sass --style compressed _sass/site.scss:resource/site.css && \
  bundle exec jekyll && \
  find _site -name "*.html" -exec tidy -config $(pwd)/tidy.conf {} \;
  
#!/usr/bin/env zsh
# File: tasks/deploy
# Build and deploy to mini.
./tasks/jekyll && \
  rsync -avz --delete _site/ mini:/Library/WebServer/hosts/metaskills.net && \
  mini sudo apachectl restart
```

<p>
  The first one, <code>task/scss</code> allowed me to author my site's CSS using <a href="http://sass-lang.com/">SASS</a> and its newer SCSS syntax, more on that later. This task simply let me publish my site and then style it later by easily saving the source SCSS file and avoiding a long running jekyll command. The second <code>task/jekyll</code> is my own long winded jekyll command. Basically it will compile my SCSS file, execute my bundled jekyll binary, then post process all my HTML files with tidy. I spent a long time coming up with <a href="https://github.com/metaskills/metaskills.net/blob/master/tidy.conf">an appropriate tidy.conf compatible with HTML5 standards.</a> Perhaps you may find it useful. Lastly, the <code>task/deploy</code> is just a simple wrapper to my own jekyll command and a final rsync command to my Mac Mini web server. I restart the web server just in case I have made changes to my <code>.htaccess</code> file.
</p>


<h2>HTML5</h2>

<p>
  A personal tech blog is a great platform to try out new things, including the semantic goodness that HTML5 offers. Frankly, it was way past time for me to learn and I found <a href="http://diveintohtml5.org/">Mark Pilgrim's - Dive Into HTML5</a> really helpful on the topic. The final structure of the blog came out nicely. A post is contained in its own <code>&lt;article&gt;</code> element with <code>&lt;header&gt;</code> and <code>&lt;footer&gt;</code> elements in between. A great way to organize comments and new elements like the <code>&lt;time&gt;</code> so you can associate a machine readable publication date to your post.
</p>

```html
<article id="post">
  <header>
    <time pubdate datetime="{{ page.date | date_to_xmlschema }}">
      <span class="day">{{ page.date | date: "%d" }}</span>
      <span class="month">{{ page.date | date: "%b" }}</span>
      <span class="year">{{ page.date | date: "%Y" }}</span>
    </time>
    <h1>{{ page.title }}</h1>
  </header>
  ...
  <footer id="disqus_thread">
    ...
  </footer>
</article>
```


<h2>Presentation - SASS</h2>

<p>
  This was the second time I have used SASS from the start of a project. The first time was almost a year ago and I must say, it has gotten much nicer with the newer SCSS format. While writing <a href="https://github.com/metaskills/metaskills.net/blob/master/_sass/site.scss">my lengthy SCSS file</a>, I found the following linear organization method very helpful. </p>

<ul>
  <li><strong>Font Declarations</strong> - With Mixins For Usage.</li>
  <li><strong>Global Variables</strong> - Grids, Colors...</li>
  <li><strong>Vendor Mixins</strong> - Gradients, Shadows, Transformations...</li>
  <li><strong>Component Mixins</strong> - Links, Navigation, Fancy Images, Comments...</li>
  <li><strong>Layout Mixins</strong> - Posts, Footer, Excerpts, Etc</li>
  <li><strong>Final Layout</strong> - Site Structure (using mixins)</li>
</ul>

<p>
  I think this is important, because it outlines a radical shift from writing raw CSS and really flexes SASS's <code>@mixin</code> and <code>@include</code> features. First it is important to point out that CSS falls into two categories, layout &amp; presentation. Layout CSS is akin to using <code>&lt;table&gt;</code> tags in the old days. It is CSS that controls the structure of your site. After that, all CSS is presentation oriented. It is there to make things look pretty.
</p>

<p>
  Back in the day, I would put my layout styles at the beginning of my stylesheet, then finish with abstract presentation classes. This works, but sometimes you find yourself deep into a CSS corner and having to whip out <code>!important</code> declarations to bail yourself out. When writing with SASS, the layout styles are now my last declarations. Why? Because you can include your earlier coded design components as a last step within the structure declarations, thereby ensuring their scope is locked down to the explicit node of your choosing. This helps you keep your layout and presentation CSS in separate and manageable chunks that have logical names while keeping your SASS file from looking like one big long procedural function. It's OO-CSS at its best!
</p>

```text
...
section#page { 
  width: $pagewidth; 
  margin: 0px auto;
  min-height: 800px;
  @include cmpnt-links;
  section#content {
    float: right;
    width: $c8width;
    padding-top: $hdrheight + 20px;
    padding-bottom: 20px;
    @include layout-excerpt;
    @include layout-post;
    @include cmpnt-flash;
    @include cmpnt-blockquote;
  }
...
```


<h2>Presentation - Simple Grid</h2>

<p>
  I have previously used CSS grids like <a href="http://960.gs/">960.gs</a> and the <a href="http://www.1kbgrid.com/">1Kb Grid</a>. However, for this project, I decided to flex SASS and my previous CSS knowledge of positioning and clearing for a leaner layout structure. These SCSS variables were enough to let me build out the new layout.
</p>

```ruby
$cols: 10;
$colwidth: 80px;
$gutterwidth: 20px;
$c1width:  $colwidth * 1;
$c2width:  ($colwidth * 2) + ($gutterwidth * 1);
$c3width:  ($colwidth * 3) + ($gutterwidth * 2);
$c4width:  ($colwidth * 4) + ($gutterwidth * 3);
$c5width:  ($colwidth * 5) + ($gutterwidth * 4);
$c6width:  ($colwidth * 6) + ($gutterwidth * 5);
$c7width:  ($colwidth * 7) + ($gutterwidth * 6);
$c8width:  ($colwidth * 8) + ($gutterwidth * 7);
$c9width:  ($colwidth * 9) + ($gutterwidth * 8);
$c10width: ($colwidth * 10) + ($gutterwidth * 9);
$pagewidth: ($colwidth * $cols) + ($gutterwidth * ($cols - 1));
```


<h2>Presentation - Pseudo Generated Content</h2>

```html
<div class="photobounding">
  <div class="tl"></div>
  <div class="tr"></div>
  <div class="bl"></div>
  <div class="br"></div>
  <div class="photoborder"></div>
  <div class="photo">
    <img src="/files/yourpicture.jpg" alt="" width="" height="" />
  </div>
</div>
```

<p>
  We have all seen code like this! It is called divitis, the overuse of span or div elements to structure and style content. I am guilty of doing this too, in fact, that code is from my old blog's CSS convention page to make a fancy border around a photo. The new MetaSkills.net takes advantage of a pseudo generated content to style complex elements and avoid this. <span class="photofancy floatr mt20 mb10 ml20"><img src="/assets/jack.png" alt="Jack Has Many Things" width="320" height="214" /></span> I would like to show you a few examples, but first, learn from the master <a href="http://twitter.com/#!/necolas">Nicolas Gallagher</a> (@necolas) in a series of articles he published for <a href="http://nicolasgallagher.com/multiple-backgrounds-and-borders-with-css2/">backgrounds/borders</a>, <a href="http://nicolasgallagher.com/pure-css-speech-bubbles/">speach bubbles</a>, and <a href="http://nicolasgallagher.com/pure-css-gui-icons/">GUI icons</a> all using pseudo generated content with CSS.
</p>

<p>
  Using these techniques, I was able to accomplish all the complex styles like my Apple TV navigation and photo fancy borders with simple semantic elements. The image to the right is only surrounded by one <code>&lt;span class="photofancy"&gt;...&lt;/span&gt;</code> element since you can not generate pseudo content from a <code>&lt;img&gt;</code> tag. I also used pseudo generated content to remove excessive layout elements too. My complex headers use them to add multiple backgrounds where only one element resides in the code. Learn this technique, it keeps things very clean.
</p>


<h2>Zepto.js - Something You Need To Watch For</h2>

<p>
  Lastly, if you have not seen the <a href="https://github.com/madrobby/zepto">zepto.js</a> project, go check it out. It bills itself as the aerogel-weight mobile javascript framework. Besides being incredibly small, I think zepto has some incredible potential when used with backbone.js. I have never used jQuery and always looked at JavaScript thru the eye's of prototype since most of my JavaScript needs require rich objects first, and access to the DOM second. I'll make another post on Zepto and why I think it is so bad ass. I use it on the new MetaSkills.net to shim up my custom pygmentize theme.
</p>


<h2>In Closing</h2>

<p>
  I am really happy with the new layout and using Jekyll to publish my static blog. I even think the old content carried over well once I tweaked it to the new CSS conventions. One particular old article that embodies all the design aspects of the new layout is <a href="/2008/9/28/jack-has_many-things">Jack has_many :things</a>. It is about a plugin I wrote on grouping ActiveRecord has_many associations. Hope you find some time to try out Jekyll too and let me know what you think of my new site!
</p>


<h2>Resources</h2>

<ul>
  <li><a href="http://techno-weenie.net/2010/6/23/you-can-let-go-now/">You Can Let Go Now</a> - Mephisto ends</li>
  <li><a href="https://github.com/mojombo/jekyll">Jekyll Project</a> - A static blog/site generator</li>
  <li><a href="https://gist.github.com/756111">Migrate Mephisto Data</a> - A gist of rails 3 model files</li>
  <li><a href="https://github.com/metaskills/metaskills.net">MetaSkills.net Source</a> - On Github.com</li>
  <li><a href="http://diveintohtml5.org/">Dive Into HTML5</a></li>
  <li><a href="http://nicolasgallagher.com/multiple-backgrounds-and-borders-with-css2/">Multiple Backgrounds &amp; Borders With CSS2</a></li>
  <li><a href="http://nicolasgallagher.com/pure-css-speech-bubbles/">Pure CSS speech bubbles</a></li>
  <li><a href="http://nicolasgallagher.com/pure-css-gui-icons/">Pure CSS GUI Icons </a></li>
  <li><a href="https://github.com/madrobby/zepto">Zepto.js</a> - The aerogel-weight mobile javascript framework</li>
  
</ul>

