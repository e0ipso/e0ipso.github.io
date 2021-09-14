---
title: 'Progressive Decoupling Made Easy'
category: drupal
categories:
  - web-development
  - drupal
tags:
  - Drupal Development
image: /assets/images/2021/links.jpg
canonical_path: https://www.lullabot.com/articles/progressive-decoupling-made-easy
---
Decoupling separates the system that stores the content from how that content is displayed on other independent systems. This can come with many benefits but also some downsides and tradeoffs. With progressive decoupling, you can get some of the benefits of decoupling while avoiding some of the downsides.

<!-- more -->
_This is a re-post of the article on the [Lullabot blog](https://www.lullabot.com/articles/progressive-decoupling-made-easy)._

<div class="video-wrapper"><iframe sandbox="allow-same-origin allow-scripts allow-popups" src="https://video.mateuaguilo.com/videos/embed/2aadbe24-2a9a-4676-9cd5-9b33a0f66736" frameborder="0" allowfullscreen></iframe></div>

<a href="https://mateuaguilo.com/assets/documents/new-standard-progressive-decoupling.pdf">Slides available here</a>.

<p>Decoupling separates the system that stores the content from how that content is displayed on other independent systems. This can come with many benefits but also some downsides and tradeoffs.</p>

<p>With progressive decoupling, you can get some of the benefits of decoupling while avoiding some of the downsides.</p>

<p>There are several ways to decouple a website progressively, but this article makes the case that widgets provide the most flexibility.</p>

<h2>What are widgets?</h2>

<p>Widgets are stand-alone JavaScript (JS) applications that are framework-agnostic and are designed to be embedded and configured by CMS editors.</p>

<p>Widgets can be vanilla JS or use frameworks like Vue or React.</p>

<h2>Why JS over server-generated HTML?</h2>

<h3>Better reactivity and interactivity</h3>

<p>The pages can be static or served from cache (very fast), and JS can be sprinkled on top.  The server can provide the unchanging parts, while the JS application adds interactivity.</p>

<p>This reduces the load on your servers while increasing website performance. You keep the benefits of built-in CMS performance tooling.</p>

<h3>Distributed delivery</h3>

<p>Different development teams can write software independently. They can publish software on the same platform without coordinating complex deployment efforts.</p>

<ul>
	<li>Teams write the JS code in isolation</li>
	<li>The browser executes the JS</li>
	<li>Different deployment pipelines and servers can be used.</li>
</ul>

![Distributed example](/assets/images/progressive-decoupling/distributedexample.png)

<p>One team works on the navigation, one team works on the main feature set, and one team works on a price calculator.</p>

<h3>Biggest talent pool</h3>

<p>According to extensive surveys, JS and TypeScript (a superset of JS) are the most commonly used languages, based on <a href="https://insights.stackoverflow.com/survey/2020#overview" target="_blank">Stackoverflow’s yearly survey</a>.</p>

<p>By building pages and experiences in JS, you can pull talent from a bigger pool. You have more options.</p>

<h3>Better developer experience</h3>

<p>Since JS is so popular, your developers can leverage many tools, services, and frameworks. Jest, Storybook, Husky, Gulp, for things like unit testing, component management, setting githooks, etc. Many services integrate with the technology.</p>

<p>Many platforms will give you better support, which leads to better workflows, which hopefully leads to better code—things like visual diffs, code quality analysis,  and code deployment. Popularity leads to a flourishing ecosystem.</p>

<p>In addition, frameworks like Vue can take care of some of the rough edges.</p>

<h3>Should we just build JS applications then?</h3>

<p>Yes and no. We still care about the content. Content is the heart of the web. You can have a great user experience, but without content, your project is doomed to fail.</p>

<p>To manage content, you need a CMS. Especially if <em>content</em> is your product or is central to your business. A CMS provides many features that are hard to build from scratch.</p>

<ul>
	<li>Managing pages and setting up URLs</li>
	<li>Users and access restrictions</li>
	<li>SEO metadata</li>
	<li>Media library</li>
	<li>Security patches</li>
	<li>Editorially controlled layouts</li>
	<li>Moderation and previews</li>
</ul>

<h2>Why widgets?</h2>

<p>We have a CMS. We know we want to use some JS. Why not put JS in our CMS templates?</p>

<p>This works. You can certainly go that route. But widgets have some advantages over JS in the template.</p>

<h3>They require no CMS deployments</h3>

<p>A developer creates a new widget in the registry, and it appears in the CMS editorial interface for embedding. No additional effort. Bug fixes and enhancements are also instantaneous.</p>

<p>Here is what a traditional deployment might look like:</p>

![JS Deploy 1](/assets/images/progressive-decoupling/js-deploy-1.png)

<ol start="1">
	<li>Develop JS app</li>
	<li>Integrate it with a CMS template (and with the content model if you want the app to receive editorial input)</li>
	<li>Deploy both in conjunction since they are coupled together</li>
	<li>Editors can expose the JS app to end-users</li>
</ol>

<p>Widgets allow you to skip the two middle steps. When you use the existing CMS integrations, development is only done in JS, and it can be deployed on its own. No need to call in a CMS developer to add new widgets or update existing widgets.</p>

<p>A widget deployment looks like this:</p>

![JS Deploy 2](/assets/images/progressive-decoupling/js-deploy-2.png)

<h3>Embedded and controlled by editors</h3>

<p>JS developers can create flexible applications that allow for tweaked experiences and configuration. A single widget can act as multiple similar widgets.</p>

<p>JS developers define the input data they expect from editors, and the CMS creates a form for the editors to input that data. This allows many instances of the same type of widget to be embedded with different configurations: different content, color palettes, external integrations, etc.</p>

<p>The following example defines a customizable button that the editor can configure.</p>

```js
settingsSchema: {
  type: 'object',
  additionalProperties: false,
  properties: {
    fields: {
      type: 'object',
      properties: {
        'button-text': {
          type: 'string',
          title: 'Button text',
          description:
            'Some random string to be displayed.',
          examples: ['I am a button', 'Please, click me'],
        },
      },
    },
  },
},
title: 'Example Widget',
status: 'stable',
```

<p>The CMS integration, which can be defined up-front, reads the definition and presents the proper form elements to the editor.</p>

![Customized button example](/assets/images/progressive-decoupling/customized-button-example.png)

<h3>Embedded anywhere</h3>

<p>Since widgets are not embedded at build time, but editorially, they can be placed anywhere. If the JS is in the template, you can’t choose, for example, to insert the JS app between two paragraphs of the body field. And changing the position would require a CMS deployment.</p>

![Body field insert](/assets/images/progressive-decoupling/body-field-insert.png)

<p>With widgets, editors can insert them anywhere.</p>

<ul>
	<li>Using layout building tools</li>
	<li>Using WYSIWYG integrations</li>
	<li>Using content modeling tools (entity reference field that points to a widget instance)</li>
	<li>Using 3rd party JavaScript</li>
</ul>

![WYSIWYG layout builder](/assets/images/progressive-decoupling/wysiwyg-layout-builder.png)

<p>And the same widget can work for any CMS. As long as the CMS subscribes to the registry and can read the schema, it can embed the JS application. When you change or fix something in the JS app, it is distributed to all CMSs. Widgets can also work in static HTML pages and Optimizely pages. Anywhere.</p>

<h2>When are widgets a good fit?</h2>

<p>Structured content is still the way to go. You don’t have to use widgets everywhere, but they are useful in several contexts.</p>

<ul>
	<li>Interacting with 3rd party APIs - reviews sites (g2crowd), commenting</li>
	<li>Interactive tools - pricing calculators, checklists saving progress</li>
	<li>Data visualizations - maps, charts of COVID data</li>
	<li>Adding some pop to a page - you can do some things with JS that may be difficult to achieve when limited to HTML and CSS</li>
</ul>

<h2>How to get started</h2>

<h3>Create a widget</h3>

<p>From a technical perspective, a widget is a function that takes a DOM id and renders JS in it. A widget can also receive arguments as HTML data.</p>

<p>Here is an example of rendering a React component:</p>

```js
  window.renderExampleWidget = function(instanceId) {
  const element = document.getElementById(instanceId);
  const title = element.getAttribute('data-button-text');
  ReactDOM.render(
    <Widget title={title} />,
    element,
  );
};
```

<p>It is very easy to port existing components and re-use them.</p>

<h3>Upload the app code</h3>

<p>The code needs to live somewhere accessible to the internet (Github pages, Amazon s3 bucket, etc.). The CMS can use this to either download the files or serve them from there. We don’t want to bundle the files within the CMS because that introduces coupling again.</p>

<h3>Publish the metadata</h3>

<p>This is the tricky part. Without the metadata, this is just another JS application in some repo.</p>

<p>We need a registry, which is just a JSON document containing the metadata about all the available apps that can be downloaded from the internet. An array of objects. This includes the “directoryUrl,” which defines exactly where the files live. You can also see the “settingsSchema” property, which defines the shape of the input data this widget will accept.</p>

```json
[
  {
    "repositoryUrl": "https://github.com/js-widgets/example-widget",
    "shortcode": "example-widget",
    "version": "v1.0.4",
    "title": "Example Widget"
    "description": "This is a widget example that showcases some of the features of the JS Widgets project."
    "directoryUrl": "https://static.mateuaguilo.com/widgets/sandbox/example-widget/v1",
    "files": [
      "css/main.css",
      "js/main.js",
      "media/logo.png",
      "thumbnail.svg"
    ],
    "availableTranslations": [
      "en",
      "de",
      "es",
      "pl",
      "pt",
      "zh-tw"
    ],
    "settingsSchema": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "fields": {"type": "object"...}
      },
    },
  },
]
```

<p>This file will need to be uploaded somewhere that is accessible via HTTP.</p>

<p>The CMS pulls that JSON object and knows about all the widgets and where to grab their assets. You’ll start seeing widgets appear in your editorial tools.</p>

![Widget list](/assets/images/progressive-decoupling/widget-list.png)

<h2>Ok…but where do I actually start?</h2>

<p>There are lots of existing tooling and examples at <a href="https://github.com/js-widgets" target="_blank">https://github.com/js-widgets</a>. It includes a registry boilerplate and catalog, widget examples, and CI/CD integration.</p>

<p>If you fork it, you’ll get a lot of nice things out of the box.</p>

<h3>Stakeholder-ready catalog</h3>

<p>The same registry that provides information to the CMS can provide information to a single-page application that is browsable and searchable. This requires zero effort. Everyone involved can see what is available: editors, developers, stakeholders, etc.</p>

<p>The catalog can also render a widget as a live sample, even if the widget requires editorial inputs. Examples utilize the “examples” key as shown in the widget definition above.</p>

![Widget catalog](/assets/images/progressive-decoupling/widget-catalog.png)

<h3>Governance like you need it</h3>

<p>All of this might seem like a governance nightmare. Do you really want JavaScript updated in a remote location and immediately deployed to your live site?</p>

<p>Of course not.</p>

<p><em>You</em> decide what registries to accept into your CMS. <em>You</em> decide what widgets and updates go into your registry. You decide who has access to change those widgets and registries.</p>

<h3>Production-ready dependencies</h3>

<p>We want these widgets as light as possible. What if there was a way not to bundle big dependencies in every single JS app? We don’t want to download React for every widget, for example.</p>

<p>Shared dependencies are possible with this paradigm. Widgets can be configured to pull certain dependencies from the parent container. This requires some Webpack configuration and telling the CMS where to find the excluded libraries. <a href="https://github.com/js-widgets/example-widget#external-dependencies" target="_blank">Read the documentation for external dependencies here</a>.</p>

<h2>Conclusion</h2>

<p>We hope this makes you excited to start taking advantage of widgets and progressive decoupling. For more videos on the specifics of setting this up, take a look at these additional videos:</p>

<ul>
	<li><a href="https://video.mateuaguilo.com/w/d1ycUz5yCj9CdEGnqKkb9X" target="_blank">Any JS app can be embedded</a></li>
	<li><a href="https://video.mateuaguilo.com/w/g2jKiesKjYSreig8abrKFh" target="_blank">The registry and the app catalog</a></li>
	<li><a href="https://video.mateuaguilo.com/w/61aG9Y8xW7m5anwupKCL84" target="_blank">Set up Progressive Decoupled Drupal</a></li>
</ul>

<small>Photo by <a href="https://unsplash.com/@polarmermaid">Anne Nygård</a> on <a href="https://unsplash.com/@polarmermaid">Unsplash</a></small>
