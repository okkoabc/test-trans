###0x6 Python Tutorial: Spidering
###0x6 Python教程：爬虫

This Python tutorial will introduce some new modules (optparse, spider) to accomplish the task of spidering a web application.  Spidering a web application is the process of enumerated linked content on a web application by following links within the web application to help build a site map.  Spidering a web application is a good use case to leverage Python to create a quick script.

You could create a crawler script by parsing the href tags on the response of a request and then create additional requests.  You can also leverage a Python module called “Spider” to do it in less lines of code:

本章Python教程将介绍一些新的模块(optparse和spider)来完成一个Web爬虫程序。爬取一个Web应用就是找出所有的网页链接，来建立一个网站地图。爬取一个网站是利用Python快速产生代码的好的例子。

你可以通过解析Web请求中的href标签然后不断请求来构建爬虫。你也可以通过Python的Spider模块和更少的代码来实现。

There are several options you can configure related to how the spider will work “myspider(b=URL.strip(), w=200, d=5, t=5)”.  This function will return two lists of child URLs and paths.  You can modify how the spider works by changing the arguments passed to the myspider function:

b — base web URL (default: None)

w — amount of resources to crawl (default: 200)

d — depth in hierarchy to crawl (default: 5)

t — number of threads (default: None)

你可以配置几个参数来决定爬虫程序如何工作--“myspider(b=URL.strip(), w=200, d=5, t=5)”。这个函数会返回一个子网页的列表和一个子网页的路径。你可以根据传入的参数来改变爬虫的工作方式:

    b - 从哪个Web链接开始(默认为空)
    w - 爬取资源链接的个数(默认200)
    d - 爬取的深度(默认为5)
    t - 线程的个数(默认为空)
    
This blog post was just a quick look into interacting with web resources by leveraging Python.  Many more advanced use cases exists for scripting web resource interaction.  Future blog posts will demonstrate some more advanced use cases by scripting attacks against web servers

这篇博文只是对Python处理Web资源的走马观花的介绍。和Web资源交互的案例有很多。以后的博文中会介绍一些使用脚本对Web服务进行攻击的高级案例。

Code snippet leveraging the Python spider module:

使用spider模块的代码段如下:

###0x7 Python Tutorial: Web Scanning and Exploitation
###0x7 Python教程：Web扫描和利用

This tutorial will demonstrate how to leverage Python to build a basic web scanner, and how to write simple exploits for web applications.  Often times an exploit Proof of Concept(PoC) code can be released before scanning and exploitation tools have checks for the vulnerability.  In this instance, it’s beneficial to spin up your own tool to check for the vulnerability across your enterprise.

本教程将展示如何利用Python来构建一个基本的网络扫描器，以及如何编写Web应用程序的exp。通常一段POC代码会在扫描和漏洞检查前被发布，在这种情况下，有利于启动你自己的工具来检查整个站点的脆弱性。

In part 0x5 we showed how to make a basic web request.  This tutorial will demonstrate two more advanced use cases for leveraging Python:

在第五章我们学习了如何进行基本的Web请求。这一章我会展示两个高级点的例子：

Checking for specific resources against a list of servers

检查一系列服务器上的特定资源

Exploiting a Local File Inclusion (LFI) vulnerability in Oracle reports.

利用一个Oracle数据库的LFI漏洞

This quick python script will accept a list of URLs pulled in from a file with switch “-i”, a list of requests to make pulled in from a file with switch “-r”, and an optional search string specified at the CLI with switch “-s”:

这段Python代码有3个参数，使用-i接受文件的URL，使用-r接受请求路径，使用-s检测是否含有CLI的字符串。

The format of the files should be as follows – File with URLs ‘http://www.google.com/’ per line, and file with requests ‘request/’ per line. Example:

reqs:

CFIDE/

admin/

tmp/

包含URL文件的格式如下，每行一个URL如(http://www.google.com/)，包含请求路径的文件格式如下，每行一个路径，如reqs文件内容:

CFIDE/

admin/

tmp/

Here is an example of invoking the script without a search term:

这有一个没有使用搜索项的脚本使用实例：

Now when making these requests, you may want to define a search term to reduce the amount of false positives you have to go through. For example, if you are requesting “/CFIDE/administrator/enter.cfm” you may want to check if “CFIDE”, or ‘.cfm’ is on the page to help reduce false positives. Below is an example of using the script with a search term:

当你构造这些请求时，可能会想要定义一个搜索项来减少出错的数量。比如，你请求了/CFIDE/administrator/enter.cfm这个路径，你可能想要看看CFIDE或者.cfm是否在页面上来减少错误。以下是一个使用搜索项的脚本实例:

As you can see only requests that contained the string ‘google’ were displayed to STDOUT. This is one example of how powerful Python can be to make a quick checker script to look for various web resources. You could take this a step further and search for version numbers and output vulnerable versions of web servers.  The full script is available at the end of the blog post.

如您所见，只有包含了‘google’字符串的页面才会显示到终端上。这个例子显示了Python在进行web资源获取上的强大威力。您可以做更进一步的完善，比如得到服务器的版本并从中找出含有漏洞的web服务器。完整的脚本代码在文章末尾。

Automating Web Application Attacks:

自动化Web应用攻击：

A few months back a security researcher NI @root posted exploit details for a Local File Inclusion (LFI) vulnerability in Oracle Reports.  At the time only PoC code existed and the exploit and vulnerability checks weren’t in any tools.  This is a perfect use case to spin up a quick script to check across some Oracle Reports web servers.  The exploit allowed you to get a local resource on the web server by sending the following request – you can specify the file or directory you are interested in after the “file:///”:

几个月前，安全研究员NI@root发布了一个Oracle的一个LFI漏洞的POC代码。那时只有POC代码和exp，而漏洞检查却没有任何工具可用。这是一个绝佳的机会，构建自己的脚本来检查使用Oracle的web服务器是否有漏洞。这个exp可以让通过file:///触发，从而得到一个web服务器上的本地文件。

Below is a quick Python script that can be invoked with the following syntax:

下面是这段Python脚本的使用语法:
