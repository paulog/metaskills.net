---
layout: post
title: PDFKit Overview & Advanced Usage
categories: 
  - ruby-rails
  - heuristics
---

<p>
  Last week I had the pleasure of rewriting 4 years of legacy PDF::Writer code to <a href="http://github.com/pdfkit/PDFKit">PDFKit</a>. Why? Well drawing pdfs in ruby using libraries like PDF::Writer is like composing a webpage with an Etch A Sketch. In short, its a damn chore that involves a bunch of code that mixes both data and presentation. Sure there are gems like <a href="http://prawn.majesticseacreature.com/">Prawn</a> that make this much easier, but nothing beats drawing your pdf code in native HTML and CSS, and that is where <a href="http://code.google.com/p/wkhtmltopdf/">wkhtmltopdf</a> comes in.
</p>

<p>
  Wkhtmltopdf is an open source project that uses the <a href="http://www.webkit.org/">WebKit</a> rendering engine to convert HTML to native PDF code. This is the muscle behind the PDFKit gem and other projects like <a href="https://github.com/mileszs/wicked_pdf">WickedPdf</a>. In this article I am only going to focus on PDFKit with Rails. But many topics will apply to both PDFKit and WickedPdf since they use wkhtmltopdf on the backside.
</p>


<h2>Installation</h2>

<p>
  Installing the PDFKit gem is a no brainer. The hard part is getting the wkhtmltopdf binaries for you platform installed. Thankfully the google project page hosts a batch of static binaries that work on just about every platform. So <a href="http://code.google.com/p/wkhtmltopdf/downloads/list">go to their download page</a> and pick the statically compiled binary that meets your needs. I highly suggest that you get the latest 0.10.0rc2 since some topics below take advantage of recent bug fixes. I have tested both the OSX and i386 on RHEL with success and the release candidate seems very production ready. I suggest placing wkhtmltopdf in <code>/usr/local/bin/wkhtmltopdf</code>.
</p>

<pre class="command">
$ which wkhtmltopdf
/usr/local/bin/wkhtmltopdf
</pre>

<h2>The Basic Requirements</h2>

<p>
  I knew that HTML to PDF generation has its drawbacks, specifically with common headers/footers and page breaks. I happily found out that wkhtmltopdf has a solution for all these problems and can layout PDF pages with pixel perfect precision. So let's skip over the <a href="https://github.com/pdfkit/PDFKit/wiki">basics</a> and get right down to using PDFKit like a pro. We are going to build out the Rails HTML/CSS layouts and templates that will solve a series of common problems.
</p>

<p>
  The major reason to use PDFKit and wkhtmltopdf is that we can use the same templating system in Rails that we use to generate other views. This means that we test our PDF view code just like any other Rails code using its built-in functional or integration test cases. Let me say that again, we can TEST our PDF code! A huge win if you have complex conditional view code. So let's get to it.
</p>


<h2>The Main PDF Layout</h2>

<p>
  Sometimes it is useful to start at the end. So the first thing we need is a new layout for all of our pdf templates to use. Here is a <a href="http://haml-lang.com/">HAML</a> file that I recommend you name <code>app/views/layouts/pdf.html.pdf.haml</code>. Did you see that name? <strong>This is important!</strong> because Rails allows us to specify template names that can service more than one mime type format. So in this case, the layout will be found when rendering both HTML and .pdf formats. 
</p>

<aside class="flash_warn">
  Somewhere along the way to Rails 3, dual mime type support was lost. Now to render a template/partial for multiple mime types you must remove them all together from the name. Thereby making the file a catch-all for any mime type. In this case, pdf.haml would work.
</aside>


```ruby
!!! 5
%html{:lang => 'en'}
  %head
    %meta{:charset => 'utf-8'}
    %meta{:name => 'pdfkit-footer_html', :content => pdf_footer_url}
    = stylesheet_link_tag "#{request.protocol}#{request.host_with_port}/stylesheets/pdf.css"
  %body
    = yield(:layout)
```

<p>
  <strong>I recommend that all PDF layouts, templates, and partials use the dual mime type naming convention.</strong> This will allow you to test your rendered HTML to PDF view code using common DOM techniques. Like <code>assert_select</code> in rails or maybe <a href="https://github.com/jnicklas/capybara">capybara's</a> <code>has_selector?</code>. So given that you may have a print action for your orders controller, you would use <code>print_orders_path(@order)</code> for your functional tests with DOM assertions and <code>print_orders_path(@order,:format=>:pdf)</code> for real world usage and/or integration tests. Both formats will render the same partials, templates and layouts if you use that naming structure.
</p>

<p>
  So that layout file is a real basic HTML5 doctype (which WebKit handles just fine) plus a few special elements. I'll cover that <code>pdfkit-footer_html</code> meta tag later on. For now, let's focus on that <code>pdf.css</code> stylesheet.
</p>


<h2>Your PDF CSS</h2>

<p>
  You might be tempted to utilize your existing site's stylesheets for a base and then use media/print techniques to override and customize your printed versions. I am of the opinion that your PDF stylesheets should be very basic and easy to layout. To this end, I highly suggest that you start with an HTML reset CSS. In the example below, I have used <a href="http://yui.yahooapis.com/3.2.0/build/cssreset/reset-min.css">Yahoo's CSS reset</a>. This makes it so that every bit of layout is under your strict control with a common rendering of no margin or padding to throw you off.
</p>

```css
/* Reset CSS. http://yui.yahooapis.com/3.2.0/build/cssreset/reset-min.css  */
html{color:#000;background:#FFF;}body,div,dl,dt,dd,ul,ol,li,h1,h2,h3,h4,h5,h6,
pre,code,form,fieldset,legend,input,textarea,p,blockquote,th,td{margin:0;padding:0;}
table{border-collapse:collapse;border-spacing:0;}fieldset,img{border:0;}address,caption,
cite,code,dfn,em,strong,th,var{font-style:normal;font-weight:normal;}li{list-style:none;}
caption,th{text-align:left;}h1,h2,h3,h4,h5,h6{font-size:100%;font-weight:normal;}
q:before,q:after{content:'';}abbr,acronym{border:0;font-variant:normal;}
sup{vertical-align:text-top;}sub{vertical-align:text-bottom;}input,textarea,
select{font-family:inherit;font-size:inherit;font-weight:inherit;}input,textarea,
select{*font-size:100%;}legend{color:#000;}

/* Your Base Foundation */
html,body { font-family: sans-serif; font-size:12px; }
h1 { font-size:18px; }
h2 { font-size:16px; }
h3 { font-size:14px; }
h4 { font-size:12px; }
strong { font-weight:900 !important; }
hr     { border:0; margin:0; padding:0; height:1px; color:#000; background-color:#000; }

/* Page Breaks */
.pb_before { page-break-before:always !important; }
.pb_after  { page-break-after:always !important; }
.pbi_avoid { page-break-inside:avoid !important; }
```

<p>
  The second section of the CSS above is the place where you can put in a few custom styles that fit your needs. In my example I set a series of header font sizes, a base sans-serif font face and an hr tag that can be used as a simple rule. Feel free to add others here like basic styles for data tables, etc. The last section of the CSS file above are page break helpers. The latest version of wkhtmltopdf never breaks text in the middle of the line anymore. So most of the time the default page break behavior will work fine. But for those situations where you need more control, these 3 CSS declarations will serve most of your needs. Let's take a look at a few examples of their usage. Full details on <a href="http://www.w3.org/TR/css3-page/">CSS paged media</a> can be found on the W3C's site.
</p>

<p>
  Use the <code>.pbi_avoid</code> class on any block level element that you want to make sure is never broken across multiple pages. A great usage would be on each element of an orders line items. It can also be used on any large page element that will certainly fit on one page, but should never be broken up. This is perfect in places where you might have measured the remaining page space before drawing said element. The <code>.pb_before</code> class will always break to a new page. I found this very useful when printing composite PDF files that combined multiple other PDF actions. So here is another HAML template that renders 3 other PDF full page partials. Each partial can be 1 to many pages. By enclosing each in a <code><div></code> tag that forces a new page break makes sure that we always start a new page when rendering each document.
</p>

```ruby
%div.pb_before
  = render :partial => 'pdf/orders/print'
%div.pb_before
  = render :partial => 'pdf/orders/invoice'
%div.pb_before
  = render :partial => 'pdf/orders/picklist'
```


<h2>Custom Headers/Footers</h2>

<p>
  PDFKit and specifically wkhtmltopdf handles common page headers and footers just wonderfully, though it did take me some time to figure it out. I'll try to spare you the same pain by detailing the process for a custom footer on each page. In this example we will expect that our custom footer will be approximately .2 inches tall with a current page number next to a total page count. 
</p>

<p>
  Remember that <code>pdfkit-footer_html</code> meta tag in the pdf layout above? If not, here it is again.
</p>

```ruby
%meta{:name => 'pdfkit-footer_html', :content => pdf_footer_url}
```

<p>
  So what is going on here? Two things really. The first is a way for PDFKit to customize the command arguments passed down to wkhtmltopdf when the page is converted. PDFKit will take any meta tag with a name prefixed using "pdfkit-" and pass down the content attribute as the value to the suffix of the name attribute. In this case <code>--foter-html http://myapp.com/pdf/footer</code> will be used as a command argument to wkhtmltopdf when rendering templates using that layout file. Note, it is important to use fully qualified URLs for header and footer arguments.
</p>

<p>
  When it comes to headers and footers, wkhtmltopdf takes the URL to an HTML page, renders it to native PDF code and embeds it automatically for you below or above your page margin. You can control the placement of these in one of two ways. The first is by adjusting the layout of the header/footer HTML page. The second is by adjusting the margin of the parent document. In my case, since I knew my footer was around .2 inches tall, I gave it's template an internal top margin of 10 pixels and told PDFKit to increase my global .5 inch page margins by .2 inches for the bottom margin using a rails initializer.
</p>

```ruby
PDFKit.configure do |config|
  config.default_options = {
    :page_size     => 'Letter',
    :margin_top    => '0.5in',
    :margin_right  => '0.5in',
    :margin_bottom => '0.7in',
    :margin_left   => '0.5in'
  }
end
```

<p>
  So now I know that whatever content I render in my <code>http://myapp.com/pdf/footer</code> page will fit just nicely on the bottom of each page. But how to generate that content and the custom page numbers? First, let's make a single pdf resource in our rails route file with a collection action for #footer. Now here is a controller for that resource with a single footer action.
</p>

```ruby
class PdfController < ApplicationController
  
  def self.perform_caching ; true ; end
  caches_page :footer
  
  def footer
    render :layout => false
  end
  
  private
  
  def perform_caching
    true
  end

end
```

<p>
  There is not much here past rendering a basic template file with no layout. All the rest is to achieve an important set of caching rules. Ideally the URL argument to <code>--footer-html</code> would be a static HTML file. However, if want to use Rails templating to render that file, it is important to cache the results. The parent document will request this URL for each page it renders, so you can see how one process could deadlock another if your were not careful. In my example above, I override ActionController's perform_caching class and instance methods so that all actions in this controller would cache. I recommend committing a cached footer html page to any source control you have for deployment.
</p>

<p>
  With that out of the way, what about the content of the footer HTML page? Again, here is a HAML template I used. This is very much like my pdf layout with one important difference, some JavaScript that is used to parse the query parameters that wkhtmltopdf tacks onto each header/footer URL request. In the example below I am only using the current page <code>page</code> and total page count <code>topage</code> params and inserting those values into to elements. For a full list of all the query parameters, consult the wkhtmltopdf expanded help page.
</p>

```ruby
!!! 5
%html{:lang => 'en'}
  %head
    %meta{:charset => 'utf-8'}
    = stylesheet_link_tag "#{request.protocol}#{request.host_with_port}/stylesheets/pdf.css"
    :javascript
      var pdfInfo = {};
      var x = document.location.search.substring(1).split('&');
      for (var i in x) { var z = x[i].split('=',2); pdfInfo[z[0]] = unescape(z[1]); }
      function getPdfInfo() {
        var page = pdfInfo.page || 1;
        var pageCount = pdfInfo.topage || 1;
        document.getElementById('pdfkit_page_current').textContent = page;
        document.getElementById('pdfkit_page_count').textContent = pageCount;
      }
  %body{:onload => 'getPdfInfo()'}
    %div#pdfkit_page_numbers
      %span Page: 
      %span#pdfkit_page_current
      %span of 
      %span#pdfkit_page_count
```

<p>
  Hopefully this helps anyone looking to use PDFKit or any system that leverages the wkhtmltopdf project. If I missed anything or can help, please leave me a comment. Thanks!
</p>



