# ASP.NET Core Markdown Support

[![NuGet Pre Release](https://img.shields.io/nuget/vpre/westwind.aspnetcore.markdown.svg)](https://www.nuget.org/packages?q=Westwind.aspnetcore.markdown)

This small package provides Markdown support for your ASP.NET Core applications. It has the following features:

*  **[Markdown Parsing Helpers](#markdown-parsing)**
    *  Generate Markdown to raw String  
      `Markdown.Parse(markdown)`
    *  Generate Markdown to HtmlString for Razor usage   
       `Markdown.ParseHtmlString(markdown)`
* **[Markdown TagHelper](#markdown-taghelper)** 
    *  Embed Markdown Content into Views and Pages
    *  Databind Markdown text
*  **[Markdown Page Processor Middleware](#markdown-page-processor-middleware)**
    *  Serve .md files as Markdown
    *  Serve extensionless URLs as Markdown
    *  Configurable Razor template to customize Markdown Page Container UI
*  [Uses the MarkDig Markdown Parser](https://github.com/lunet-io/markdig)


Related links:

* [Markdown TagHelper Blog Post](https://weblog.west-wind.com/posts/2018/Mar/23/Creating-an-ASPNET-Core-Markdown-TagHelper-and-Parser)
* [Markdown Page Handler Middleware Blog Post](https://weblog.west-wind.com/posts/2018/Apr/18/Creating-a-generic-Markdown-Page-Handler-in-ASPNET-Core)

## Installing the NuGet Package
You can install the package [from NuGet](https://www.nuget.org/packages/Westwind.AspNetCore.Markdown/) in Visual Studio or from the command line:

```ps
PM> install-package westwind.aspnetcore.markdown
```

or the `dotnet` command line:

```ps
dotnet add package westwind.aspnetcore.markdown
```

## Markdown Parsing Helpers
At it's simplest this component provides Markdown parsing that you can use to convert Markdown text to HTML either inside of application code, or inside of Razor expressions.

### Markdown to String

```cs
string html = Markdown.Parse(markdownText)
```

### Markdown to Razor Html String

```cs
<div>@Markdown.ParseHtmlString(Model.ProductInfoMarkdown)</div>
```

### MarkDig Pipeline Configuration
This component uses the MarkDig Markdown Parser which allows for explicit feature configuration via many of its built-in extensions. The default configuration enables the most commonly used Markdown features and defaults to Github Flavored Markdown for most settings.

If you need to customize what features are supported you can override the pipeline creation explicitly in the `Startup.ConfigureServices()` method:

```cs
services.AddMarkdown(config =>
{
    // Create custom MarkdigPipeline 
    // using MarkDig; for extension methods
    config.ConfigureMarkdigPipeline = builder =>
    {
        builder.UseEmphasisExtras(Markdig.Extensions.EmphasisExtras.EmphasisExtraOptions.Default)
            .UsePipeTables()
            .UseGridTables()                        
            .UseAutoIdentifiers(AutoIdentifierOptions.GitHub) // Headers get id="name" 
            .UseAutoLinks() // URLs are parsed into anchors
            .UseAbbreviations()
            .UseYamlFrontMatter()
            .UseEmojiAndSmiley(true)                        
            .UseListExtras()
            .UseFigures()
            .UseTaskLists()
            .UseCustomContainers()
            .UseGenericAttributes();
    };
});
```

When set this configuration is used every time the Markdown parser instance is created instead of the default behavior.

## Markdown TagHelper
The Markdown TagHelper allows you to embed static Markdown content into a `<markdown>` tag. The TagHelper supports both embedded content, or an attribute based value assignment or model binding via the `markdown` attribute.

To get started with the Markdown TagHelper you need to do the following:

* Register TagHelper in `_ViewImports.cshtml`
* Place a `<markdown>` TagHelper on the page
* Use `Markdown.ParseHtmlString()` in Razor Page expressions
* Rock on!

After installing the NuGet package **you have to register** the tag helper so MVC can find it. The easiest way to do this is to add it to the `_ViewImports.cshtml` file in your `Views\Shared` folder for MVC or the root for your `Pages` folder.

```html
@addTagHelper *, Westwind.AspNetCore.Markdown
```

### Literal Markdown Content
To use the literal content control you can simply place your Markdown text between the opening and closing `<markdown>` tags:

```html
<markdown>
    #### This is Markdown text inside of a Markdown block

    * Item 1
    * Item 2
 
    ### Dynamic Data is supported:
    The current Time is: @DateTime.Now.ToString("HH:mm:ss")

    ```cs
    // this c# is a code block
    for (int i = 0; i < lines.Length; i++)
    {
        line1 = lines[i];
        if (!string.IsNullOrEmpty(line1))
            break;
    }
    ```
</markdown>
```

The TagHelper turns the Markdown text into HTML, in place of the TagHelper content.

> ##### Razor Expression Evaluation
> Note that Razor expressions in the markdown content are supported - as the `@DateTime.Now.ToString()` expression in the example -  and are expanded **before** the content is parsed by the TagHelper. This means you can embed dynamic values into the markdown content which gives you most of the flexibilty of Razor code now in Markdown. Embedded expressions **are not automatically HTML encoded** as they are embedded into the Markdown. Additionally you can also generate additional Markdown as part of a Razor expression to make the Markdown even more dynamic.

### Markdown Attribute and DataBinding
In addition to the content you can also bind to the `markdown` attribute which allows for programmatic assignment and databinding.

```
@model MarkdownPageModel
@{
    Model.MarkdownText = "This is some **Markdown**!";
}

<markdown markdown="Model.MarkdownText" />
```

The `markdown` attribute accepts binding expressions so you can bind Markdown for display from model values or other expressions easily.

### NormalizeWhiteSpace
Markdown is sensitive to leading spaces and given that you're likely to enter Markdown into the literal TagHelper in a code editor there's likely to be a leading block of white space. Markdown treats leading whitespace as significant - 4 spaces or a tab indicate a code block so if you have:

```html
<markdown>
    #### This is Markdown text inside of a Markdown block

    * Item 1
    * Item 2
 
    ### Dynamic Data is supported:
    The current Time is: @DateTime.Now.ToString("HH:mm:ss")

</markdown>
```

without special handling Markdown would interpret the entire markdown text as a single code block.

By default the TagHelper sets `normalize-whitespace="true"` which automatically strips common whitespace to all lines from the code block. Note that this is based on the formatting of the first non-blank line of code and works only if all code lines start with the same formatted whitespace.

Optionally you can also force justify your code and turn the setting to `false`:

```html
<markdown normalize-whitespace="false">
#### This is Markdown text inside of a Markdown block

* Item 1
* Item 2

### Dynamic Data is supported:
The current Time is: @DateTime.Now.ToString("HH:mm:ss")

</markdown>
```

This also works, but is hard to maintain in some code editors due to auto-code reformatting.

## Markdown Page Processor Middleware
The Markdown middleware allows you drop `.md` files into a configured folder and have that folder parsed directly from disk. The middleware merges the Markdown into a pre-configured Razor template you provide so your Markdown text can be rendered in the proper UI context of your site chrome. 

To use this feature you need to do the following:

* Use `AddMarkdown()` to **configure** the page processing
* Use `UseMarkdown()` to **hook up** the middleware
* Create a Markdown View Template (default is: `~/Views/__MarkdownPageTemplate.cshtml`)
* Create `.md` files for your content
* Rock on!

Note that the default template location can be customized for each folder and it can live anywhere including the `Pages` folder.

### Startup Configuration
As with any middleware components you need to configure the Markdown middleware and hook it up for processing which is a two step process.

First you need to call `AddMarkdown()` to configure the Markdown processor. You need to specify the folders that the processor is supposed to work on.

At it's simplest you can just do:

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddMarkdown(config =>
    {
        // just add a folder as is
        config.AddMarkdownProcessingFolder("/docs/");
        
        services.AddMvc();
    }
}
```

This configures the Markdown processor for default behavior which handles `.md` files and extensionless Urls in the `/docs/` folder as Markdown files (if they exist) and it assumes a default Razor Markdown host template at `~/Views/__MarkdownPageTemplate.cshtml`. 

All of these values can be customized with additional configuration options:

```cs
services.AddMarkdown(config =>
{
    // Simplest: Use all default settings - usually all you need
    config.AddMarkdownProcessingFolder("/docs/", "~/Pages/__MarkdownPageTemplate.cshtml");
    
    // Customized Configuration: Set FolderConfiguration options
    var folderConfig = config.AddMarkdownProcessingFolder("/posts/", "~/Pages/__MarkdownPageTemplate.cshtml");

    // Optional configuration settings
    folderConfig.ProcessExtensionlessUrls = true;  // default
    folderConfig.ProcessMdFiles = true;            // default

    // Optional pre-processing
    folderConfig.PreProcess = (model, controller) =>
    {
        //controller.ViewBag.Model = new MyCustomModel();
    };
    
    // optional custom MarkdigPipeline (using MarkDig; for extension methods)
    config.ConfigureMarkdigPipeline = builder =>
    {
        builder.UseEmphasisExtras(Markdig.Extensions.EmphasisExtras.EmphasisExtraOptions.Default)
            .UsePipeTables()
            .UseGridTables()                        
            .UseAutoIdentifiers(AutoIdentifierOptions.GitHub) // Headers get id="name" 
            .UseAutoLinks() // URLs are parsed into anchors
            .UseAbbreviations()
            .UseYamlFrontMatter()
            .UseEmojiAndSmiley(true)                        
            .UseListExtras()
            .UseFigures()
            .UseTaskLists()
            .UseCustomContainers()
            .UseGenericAttributes();
    };
}   
```

There are additional options including the ability to hook in a pre-processor that's fired on every controller hit. In the example I set a custom model to the ViewBag that the template can potentially pick up and work with. For applications you might have a stock View model that provides access rights and other user logic that needs to fire to access the page and display the view. Using the `PreProcess` Action hook you can run just about any pre-processing logic and get values into the View if necessary via the `Controller.Viewbag`.

In addition to the configuration you also need to hook up the Middleware in the `Configure()` method:

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{  
    ...
    app.UseMarkdown();
    
    app.UseStaticFiles();
    app.UseMvc();
}
```        

The `UseMarkdown()` method hooks up the middleware into the pipeline.

Note that both `ConfigureServices()` and `Configure()` are required to reference the MVC middleware - `services.AddMvc()` and `app.UseMvc()` respectively - as the middleware relies on MVC and Razor to render the Razor host view template.

### Create a Markdown Page View Template
Markdown is just an HTML fragment, not a complete document, so a host template is required into which the rendered Markdown is embedded. To accomplish this the middleware uses a well-known MVC controller endpoint that loads the configured Razor view template that embeds the Markdown text.

The middleware reads in the Markdown file from disk, and then uses a generic MVC controller method call the specified template to render the page containing your Markdown text as the content. The template is passed a `MarkdownModel` that includes `RenderedMarkdown` and `Title` properties.

Typically the template page is pretty simple and only contains the rendered Markdown plus a reference to a `_Layout` page, but what goes into this template is really up to you.

The actual rendered Markdown content renders into HTML is pushed into the page via  `ViewBag.RenderedMarkdown`, which is an `HtmlString` instance that contains the rendered HTML. 

A minimal template page can do something like this:

```html
@model Westwind.AspNetCore.Markdown.MarkdownModel
@{
    ViewBag.Title = Model.Title;
    Layout = "_Layout";
}
<div style="margin-top: 40px;">
    @Model.RenderedMarkdown
</div>
```

This template can be a self contained file, or as I am doing here, it can reference a `_layout` page so that the  layout matches the rest of your site.

A more complete template might also add a code highlighter ([highlightJs](https://highlightjs.org/https://highlightjs.org/) here) and custom styling something more along the lines of this:

```html
@model Westwind.AspNetCore.Markdown.MarkdownModel
@{
    Layout = "_Layout";
}
@section Headers {
    <style>
        h3 {
            margin-top: 50px;
            padding-bottom: 10px;
            border-bottom: 1px solid #eee;
        }
        /* vs2015 theme specific*/
        pre {
            background: #1E1E1E;
            color: #eee;
            padding: 0.7em !important;
            overflow-x: auto;
            white-space: pre;
            word-break: normal;
            word-wrap: normal;
        }

            pre > code {
                white-space: pre;
            }
    </style>
}
<div style="margin-top: 40px;">
    @Model.RenderedMarkdown
</div>

@section Scripts {
    <script src="~/lib/highlightjs/highlight.pack.js"></script>
    <link href="~/lib/highlightjs/styles/vs2015.css" rel="stylesheet" />
    <script>
        setTimeout(function () {
            var pres = document.querySelectorAll("pre>code");
            for (var i = 0; i < pres.length; i++) {
                hljs.highlightBlock(pres[i]);
            }
        });

    </script>
}
```

### Create your Markdown Pages
Finally you need to create your Markdown pages in the folders you configured. Assume for a minute that you hooked up a `Posts` folder for Markdown Processing. The folder refers to the `\wwwroot\Posts` folder in your ASP.NET Core project. You can now create a new markdown file in that folder or any subfolder below it. I'm going to pretend I create blog post in a typical data folder structure. 

For example:

```
wwwroot/posts/2018/03/23/MarkdownTagHelper.md
```

![](ProjectFolderLayout.png)

I can now access this post using either:

```
http://localhost:59805/posts/2018/03/23/MarkdownTagHelper.md
```

or if extensionless URLs are configured:

```
http://localhost:59805/posts/2018/03/23/MarkdownTagHelper
```

The screenshot below shows the output of this page which is a Markdown blog post I simply copied into a folder along with a couple of support images. This is in a stock ASP.NET Core MVC project without any other changes save a few label updates:

![](MarkdownRenderedPage.png)

Voila generically rendered Markdown content from .md files on disk.


## Markdown Styling
A Markdown parser **only converts Markdown to HTML**, it does nothing for styling or how that rendered HTML content is displayed. It's up to the hosting application to provide the styling applied to the rendered HTML. For the most part HTML 'just works'. For example the stock ASP.NET Core Bootstrap templates render most markdown text nicely as you would expect.

### Syntax Highlighting
One thing that you usually will need to add is code syntax highlighting support. Syntax highlighting can be accomplished by using a JavaScript code highlighting addin. You can check out the [Markdown.cshtml sample page](https://github.com/RickStrahl/Westwind.AspNetCore/blob/master/SampleWeb/Pages/Markdown.cshtml) to see how to use [HiLightJs ](https://highlightjs.org/).

The following adds basic syntax coloring support using a preconfigured package of languages.
 
```html
@section Scripts {
    <script src="~/lib/highlightjs/highlight.pack.js"></script>
    <link href="~/lib/highlightjs/styles/vs2015.css" rel="stylesheet" />
    <script>
        setTimeout(function () {
            var pres = document.querySelectorAll("pre>code");
            for (var i = 0; i < pres.length; i++) {
                hljs.highlightBlock(pres[i]);
            }
        });

    </script>
}
```

> #### highlightJs from CDN
> The provided highlightJs package includes a custom compiled set of languages that I use most commonly. You can download and create your own package. You can also pick up HighlightJs directly off a CDN with the default language pack which may or may not provide the languages you need to display:
> ```
> <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/highlight.min.js"></script>
> <link href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/styles/vs2015.min.css" rel="stylesheet" />
> ``` 
