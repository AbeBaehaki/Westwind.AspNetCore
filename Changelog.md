# Westwind.AspNetCore Change Log
<small>[Nuget](https://www.nuget.org/packages/Westwind.AspNetCore/) &bull; [Github](https://github.com/RickStrahl/Westwind.AspNetCore)</small>

### Version 3.15

* **Add Markdown TagHelper**  
Added a `<markdown />` TagHelper which allows embedding of static Markdown content into a page which is parsed into HTML at runtime. Also includes a Markdown Parser using `Markdown.Parse()` and `Markdown.ParseHtmlString()`. Uses the MarkDig Markdown Parser. Markdown features live in a separate NuGet package `Westwind.AspNetCore.Markdown`.

* **Add Markdown Parser**  
You can use the `Markdown.Parse()` and `Markdown.ParseHtmlString()` methods to render Markdown to HTML in code and Razor pages respectively.

* **AppUser Updates**   
Add additional functions to help with ClaimsPrincipal retrieval.


* **ErrorDisplay Tag Helper Updates**  
Fix Font-Awesome icon display for warning and errors. Fix `UnhandledApiExceptionFilter` bug with invalid declaration.