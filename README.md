# Web_scraping_BestBooksWebsite_GoodReads_PYTHON_ipynb

%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import time
from bs4 import BeautifulSoup
import requests
To fetch a webpage's content, we can simply use the get() function within the requests library:

url = "https://www.npr.org/2018/11/05/664395755/what-if-the-polls-are-wrong-again-4-scenarios-for-what-might-happen-in-the-elect"
response = requests.get(url) # you can use any URL that you wish
The response variable has many highly useful attributes, such as:

status_code
text
content
Let's try each of them!

response.status_code
response.status_code
You should have received a status code of 200, which means the page was successfully found on the server and sent to receiver (aka client/user/you). Again, you can click here for a full list of status codes.

response.text
response.text
Holy moly! That looks awful. If we use our browser to visit the URL, then right-click the page and click 'View Page Source', we see that it is identical to this chunk of glorious text.

response.content
response.content
What?! This seems identical to the .text field. However, the careful eye would notice that the very 1st characters differ; that is, .content has a b' character at the beginning, which in Python syntax denotes that the data type is bytes, whereas the .text field did not have it and is a regular String.

Ok, so that's great, but how do we make sense of this text? We could manually parse it, but that's tedious and difficult. As mentioned, BeautifulSoup is specifically designed to parse this exact content (any webpage content).

BEAUTIFUL SOUP
title (property of NBC)

The documentation for BeautifulSoup is found here.

A BeautifulSoup object can be initialized with the .content from request and a flag denoting the type of parser that we should use. For example, we could specify html.parser, lxml, etc documentation here. Since we are interested in standard webpages that use HTML, let's specify the html.parser:

soup = BeautifulSoup(response.content, "html.parser")
soup
Alright! That looks a little better; there's some whitespace formatting, adding some structure to our content! HTML code is structured by <tags>. Every tag has an opening and closing portion, denoted by < > and </ >, respectively. If we want just the text (not the tags), we can use:

soup.get_text()
There's some tricky Javascript still nesting within it, but it definitely cleaned up a bit. On other websites, you may find even clearer text extraction.

As detailed in the BeautifulSoup documentation, the easiest way to navigate through the tags is to simply name the tag you're interested in. For example:

soup.head # fetches the head tag, which ecompasses the title tag
Usually head tags are small and only contain the most important contents; however, here, there's some Javascript code. The title tag resides within the head tag.

soup.title # we can specifically call for the title tag
This result includes the tag itself. To get just the text within the tags, we can use the .name property.

soup.title.string
We can navigate to the parent tag (the tag that encompasses the current tag) via the .parent attribute:

soup.title.parent.name
3. Parse the page with Beautiful Soup
In HTML code, paragraphs are often denoated with a <p> tag.

soup.p
This returns the first paragraph, and we can access properties of the given tag with the same syntax we use for dictionaries and dataframes:

soup.p['class']
In addition to 'paragraph' (aka p) tags, link tags are also very common and are denoted by <a> tags

soup.a
It is called the a tag because links are also called 'anchors'. Nearly every page has multiple paragraphs and anchors, so how do we access the subsequent tags? There are two common functions, .find() and .find_all().

soup.find('title')
soup.find_all('title')
Here, the results were seemingly the same, since there is only one title to a webpage. However, you'll notice that .find_all() returned a list, not a single item. Sure, there was only one item in the list, but it returned a list. As the name implies, find_all() returns all items that match the passed-in tag.

soup.find_all('a')
Look at all of those links! Amazing. It might be hard to read but the href portion of an a tag denotes the URL, and we can capture it via the .get() function.

for link in soup.find_all('a'): # we could optionally pass the href=True flag .find_all('a', href=True)
    print(link.get('href'))
Many of those links are relative to the current URL (e.g., /section/news/).

paragraphs = soup.find_all('p')
paragraphs
If we want just the paragraph text:

for pa in paragraphs:
    print(pa.get_text())
Since there are multiple tags and various attributes, it is useful to check the data type of BeautifulSoup objects:

type(soup.find('p'))
Since the .find() function returns a BeautifulSoup element, we can tack on multiple calls that continue to return elements:

soup.find('p')
soup.find('p').find('a')
soup.find('p').find('a').attrs['href'] # att
soup.find('p').find('a').text
That doesn't look pretty, but it makes sense because if you look at what .find('a') returned, there is plenty of whitespace. We can remove that with Python's built-in .strip() function.

soup.find('p').find('a').text.strip()
NOTE: above, we accessed the attributes of a link by using the property .attrs. .attrs takes a dictionary as a parameter, and in the example above, we only provided the key, not a value, too. That is, we only cared that the <a> tag had an attribute named href (which we grabbed by typing that command), and we made no specific demands on what the value must be. In other words, regardless of the value of href, we grabbed that element. Alternatively, if you inspect your HTML code and notice select regions for which you'd like to extract text, you can specify it as part of the attributes, too!

For example, in the full response.text, we see the following line:

<header class="npr-header" id="globalheader" aria-label="NPR header">

Let's say that we know that the information we care about is within tags that match this template (i.e., class is an attribute, and its value is 'npr-header').

soup.find('header', attrs={'class':'npr-header'})
This matched it! We could then continue further processing by tacking on other commands:

soup.find('header', attrs={'class':'npr-header'}).find_all("li") # li stands for list items
This returns all of our list items, and since it's within a particular header section of the page, it appears they are links to menu items for navigating the webpage. If we wanted to grab just the links within these:

menu_links = set()
for list_item in soup.find('header', attrs={'class':'npr-header'}).find_all("li"):
    for link in list_item.find_all('a', href=True):
        menu_links.add(link)
menu_links # a unique set of all the seemingly important links in the header
TAKEAWAY LESSON
The above tutorial isn't meant to be a study guide to memorize; its point is to show you the most important functionaity that exist within BeautifulSoup, and to illustrate how one can access different pieces of content. No two web scraping tasks are identical, so it's useful to play around with code and try different things, while using the above as examples of how you may navigate between different tags and properties of a page. Don't worry; we are always here to help when you get stuck!

String formatting
As we parse webpages, we may often want to further adjust and format the text to a certain way.

For example, say we wanted to scrape a polical website that lists all US Senators' name and office phone number. We may want to store information for each senator in a dictionary. All senators' information may be stored in a list. Thus, we'd have a list of dictionaries. Below, we will initialize such a list of dictionary (it has only 3 senators, for illustrative purposes, but imagine it contains many more).

# this is a bit clumsy of an initialization, but we spell it out this way for clarity purposes
# NOTE: imagine the dictionary were constructed in a more organic manner
senator1 = {"name":"Lamar Alexander", "number":"555-229-2812"}
senator2 = {"name":"Tammy Baldwin", "number":"555-922-8393"}
senator3 = {"name":"John Barrasso", "number":"555-827-2281"}
senators = [senator1, senator2, senator3]
print(senators)
In the real-world, we may not want the final form of our information to be in a Python dictionary; rather, we may need to send an email to people in our mailing list, urging them to call their senators. If we have a templated format in mind, we can do the following:

email_template = """Please call {name} at {number}"""
for senator in senators:
    print(email_template.format(**senator))
Please visit here for further documentation

Alternatively, one can also format their text via the f'-strings property. See documentation here. For example, using the above data structure and goal, one could yield identical results via:

for senator in senators:
    print(f"Please call {senator['name']} at {senator['number']}")
Additionally, sometimes we wish to search large strings of text. If we wish to find all occurrences within a given string, a very mechanical, procedural way of doing it would be to use the .find() function in Python and to repeatedly update the starting index from which we are looking.

Regular Expressions
A way more suitable and powerful way is to use Regular Expressions, which is a pattern matching mechanism used throughout Computer Science and programming (it's not just specific to Python). A tutorial on Regular Expressions (aka regex) is beond this lab, but below are many great resources that we recommend, if you are interested in them (could be very useful for a homework problem):

https://docs.python.org/3.3/library/re.html
https://regexone.com
https://docs.python.org/3/howto/regex.html.
Additonal Python/Homework Comment
In Homework #1, we ask you to complete functions that have signatures with a syntax you may not have seen before:

def create_star_table(starlist: list) -> list:

To be clear, this syntax merely means that the input parameter must be a list, and the output must be a list. It's no different than any other function, it just puts a requirement on the behavior of the function.

It is typing our function. Please see this documention if you have more questions.

Walkthrough Example (of Web Scraping)
We're going to see the structure of Goodread's best books list (NOTE: Goodreads is described a little more within the other Lab2_More_Pandas.ipynb notebook). We'll use the Developer tools in chrome, safari and firefox have similar tools available. To get this page we use the requests module. But first we should check if the company's policy allows scraping. Check the robots.txt to find what sites/elements are not accessible. Please read and verify.



url="https://www.npr.org/2018/11/05/664395755/what-if-the-polls-are-wrong-again-4-scenarios-for-what-might-happen-in-the-elect"
response = requests.get(url)
# response.status_code
# response.content

# Beautiful Soup (library) time!
soup = BeautifulSoup(response.content, "html.parser")
# print(soup)
# print(soup.prettify())
soup.find("title")

    # Q1: how do we get the title's text?
# soup.find("title").string
    # Q2: how do we get the webpage's entire content?
# soup.get_text()
URLSTART="https://www.goodreads.com"
BESTBOOKS="/list/show/1.Best_Books_Ever?page="
url = URLSTART+BESTBOOKS+'1'
print(url)
page = requests.get(url)
We can see properties of the page. Most relevant are status_code and text. The former tells us if the web-page was found, and if found , ok. (See lecture notes.)

page.status_code # 200 is good
36
200
page.text[:5000]
37
'<!DOCTYPE html>\n<html class="desktop withSiteHeaderTopFullImage\n">\n<head>\n  <title>Best Books Ever (94932 books)</title>\n\n<meta content=\'93,439 books based on 229859 votes: The Hunger Games by Suzanne Collins, Harry Potter and the Order of the Phoenix by J.K. Rowling, Pride and Prejudice b...\' name=\'description\'>\n<meta content=\'telephone=no\' name=\'format-detection\'>\n<link href=\'https://www.goodreads.com/list/show/1.Best_Books_Ever\' rel=\'canonical\'>\n\n\n\n    <script type="text/javascript"> var ue_t0=window.ue_t0||+new Date();\n </script>\n  <script type="text/javascript">\n    var ue_mid = "A1PQBFHBHS6YH1";\n    var ue_sn = "www.goodreads.com";\n    var ue_furl = "fls-na.amazon.com";\n    var ue_sid = "727-5034609-6932727";\n    var ue_id = "GQKRR3AQ2FANQDXA7AAG";\n\n    (function(e){var c=e;var a=c.ue||{};a.main_scope="mainscopecsm";a.q=[];a.t0=c.ue_t0||+new Date();a.d=g;function g(h){return +new Date()-(h?0:a.t0)}function d(h){return function(){a.q.push({n:h,a:arguments,t:a.d()})}}function b(m,l,h,j,i){var k={m:m,f:l,l:h,c:""+j,err:i,fromOnError:1,args:arguments};c.ueLogError(k);return false}b.skipTrace=1;e.onerror=b;function f(){c.uex("ld")}if(e.addEventListener){e.addEventListener("load",f,false)}else{if(e.attachEvent){e.attachEvent("onload",f)}}a.tag=d("tag");a.log=d("log");a.reset=d("rst");c.ue_csm=c;c.ue=a;c.ueLogError=d("err");c.ues=d("ues");c.uet=d("uet");c.uex=d("uex");c.uet("ue")})(window);(function(e,d){var a=e.ue||{};function c(g){if(!g){return}var f=d.head||d.getElementsByTagName("head")[0]||d.documentElement,h=d.createElement("script");h.async="async";h.src=g;f.insertBefore(h,f.firstChild)}function b(){var k=e.ue_cdn||"z-ecx.images-amazon.com",g=e.ue_cdns||"images-na.ssl-images-amazon.com",j="/images/G/01/csminstrumentation/",h=e.ue_file||"ue-full-11e51f253e8ad9d145f4ed644b40f692._V1_.js",f,i;if(h.indexOf("NSTRUMENTATION_FIL")>=0){return}if("ue_https" in e){f=e.ue_https}else{f=e.location&&e.location.protocol=="https:"?1:0}i=f?"https://":"http://";i+=f?g:k;i+=j;i+=h;c(i)}if(!e.ue_inline){if(a.loadUEFull){a.loadUEFull()}else{b()}}a.uels=c;e.ue=a})(window,document);\n\n    if (window.ue && window.ue.tag) { window.ue.tag(\'list:show:signed_out\', ue.main_scope);window.ue.tag(\'list:show:signed_out:desktop\', ue.main_scope); }\n  </script>\n\n  <!-- * Copied from https://info.analytics.a2z.com/#/docs/data_collection/csa/onboard */ -->\n<script>\n  //<![CDATA[\n    !function(){function n(n,t){var r=i(n);return t&&(r=r("instance",t)),r}var r=[],c=0,i=function(t){return function(){var n=c++;return r.push([t,[].slice.call(arguments,0),n,{time:Date.now()}]),i(n)}};n._s=r,this.csa=n}();\n    \n    if (window.csa) {\n      window.csa("Config", {\n        "Application": "GoodreadsMonolith",\n        "Events.SushiEndpoint": "https://unagi.amazon.com/1/events/com.amazon.csm.csa.prod",\n        "Events.Namespace": "csa",\n        "CacheDetection.RequestID": "GQKRR3AQ2FANQDXA7AAG",\n        "ObfuscatedMarketplaceId": "A1PQBFHBHS6YH1"\n      });\n    \n      window.csa("Events")("setEntity", {\n        session: { id: "727-5034609-6932727" },\n        page: {requestId: "GQKRR3AQ2FANQDXA7AAG", meaningful: "interactive"}\n      });\n    }\n    \n    var e = document.createElement("script"); e.src = "https://m.media-amazon.com/images/I/41mrkPcyPwL.js"; document.head.appendChild(e);\n  //]]>\n</script>\n\n\n          <script type="text/javascript">\n        if (window.Mobvious === undefined) {\n          window.Mobvious = {};\n        }\n        window.Mobvious.device_type = \'desktop\';\n        </script>\n\n\n  \n<script src="https://s.gr-assets.com/assets/webfontloader-2d88cff0adfd51bd1d253f792b05e614.js"></script>\n<script>\n//<![CDATA[\n\n  WebFont.load({\n    classes: false,\n    custom: {\n      families: ["Lato:n4,n7,i4", "Merriweather:n4,n7,i4"],\n      urls: ["https://s.gr-assets.com/assets/gr/fonts-e256f84093cc13b27f5b82343398031a.css"]\n    }\n  });\n\n//]]>\n</script>\n\n  <link rel="stylesheet" media="all" href="https://s.gr-assets.com/assets/goodreads-ecf9285fef7194f28b662030726252f7.css" />\n\n    <style type="text/css" media="screen">\n    .bigTabs {\n      margin-bottom: 10px;\n    }\n\n    .list_read{\n      background-color: #D7D2C4;\n      float: left;\n    }\n  </style>\n\n\n  <link rel="stylesheet" media="screen" href="https://s.gr-assets.com/assets/common_images-f5630939f2056b14f661a80fa8503dca.css" />\n\n  <script src="https://s.gr-assets.com/assets/desktop/libraries-c07ee2e4be9ade4a64546b3ec60b523b.js"></script>\n  <script src="https://s.gr-assets.com/assets/application-0f595246102859de57f612729050f678.js"></script>\n\n    <script>\n  //<![CDATA[\n    var gptAdSlots = gptAdSlots || [];\n    var googletag = googletag || {};\n    googletag.cmd = googletag.cmd || [];\n    (function() {\n      var gads = document.createElement("script");\n      gads.async = true;\n      gads.type = "text/javascript";\n      var useSSL = "https:" == document.location.protocol;\n      gads.src = (useSSL ? "https:" : "http:") +\n      "//securepubads.g.doubleclick.net/tag/js/gpt.js";\n      var node = document.get'
Let us write a loop to fetch 2 pages of "best-books" from goodreads. Notice the use of a format string. This is an example of old-style python format strings

URLSTART="https://www.goodreads.com"
BESTBOOKS="/list/show/1.Best_Books_Ever?page="
for i in range(1,3):
    bookpage=str(i)
    stuff=requests.get(URLSTART+BESTBOOKS+bookpage)
    # filetowrite="files/page"+ '%02d' % i + ".html"
    filetowrite="page"+ '%02d' % i + ".html"

    print("FTW", filetowrite)
    fd=open(filetowrite,"w")
    fd.write(stuff.text)
    fd.close()
    # f = open("page01.html", "w")
    # f.write("Now the file has more content!")
    # f.close()
    time.sleep(2)
FTW page01.html
---------------------------------------------------------------------------

UnicodeEncodeError                        Traceback (most recent call last)

Cell In [38], line 11
      9 print("FTW", filetowrite)
     10 fd=open(filetowrite,"w")
---> 11 fd.write(stuff.text)
     12 fd.close()
     13 # f = open("page01.html", "w")
     14 # f.write("Now the file has more content!")
     15 # f.close()

File ~\AppData\Local\Programs\Python\Python310\lib\encodings\cp1252.py:19, in IncrementalEncoder.encode(self, input, final)
     18 def encode(self, input, final=False):
---> 19     return codecs.charmap_encode(input,self.errors,encoding_table)[0]

UnicodeEncodeError: 'charmap' codec can't encode character '\u25be' in position 16100: character maps to <undefined>

2. Parse the page, extract book urls
Notice how we do file input-output, and use beautiful soup in the code below. The with construct ensures that the file being read is closed, something we do explicitly for the file being written. We look for the elements with class bookTitle, extract the urls, and write them into a file

bookdict={}
for i in range(1,3):
    books=[]
    stri = '%02d' % i
    # filetoread="files/page"+ stri + '.html'
    filetoread="page"+ stri + '.html'
    print("FTW", filetoread)
    with open(filetoread) as fdr:
        data = fdr.read()
    soup = BeautifulSoup(data, 'html.parser')
    for e in soup.select('.bookTitle'):
        books.append(e['href'])
    print(books[:10])
    bookdict[stri]=books
    print(bookdict)
    fd=open("list"+stri+".txt","w")
    fd.write("\n".join(books))
    fd.close()
Here is George Orwell's 1984

bookdict['02'][0]
Lets go look at the first URLs on both pages



3. Parse a book page, extract book properties
Ok so now lets dive in and get one of these these files and parse them.

furl=URLSTART+bookdict['02'][0]
furl


fstuff=requests.get(furl)
print(fstuff.status_code)
#d=BeautifulSoup(fstuff.text, 'html.parser')
# try this to take care of arabic strings
d = BeautifulSoup(fstuff.text, 'html.parser', from_encoding="utf-8")
d.select("meta[property='og:title']")[0]['content']
Lets get everything we want...

d
#d=BeautifulSoup(fstuff.text, 'html.parser', from_encoding="utf-8")
print(
"title", d.select_one("meta[property='og:title']")['content'],"\n",
# "isbn", d.select("meta[property='books:isbn']")[0]['content'],"\n",
"type", d.select("meta[property='og:type']")[0]['content'],"\n",
# "author", d.select("meta[property='books:author']")[0]['content'],"\n",
#"average rating", d.select_one("span.average").text,"\n",
# "ratingCount", d.select("meta[itemprop='ratingCount']")[0]["content"],"\n"
#"reviewCount", d.select_one("span.count")["title"]
)
Ok, now that we know what to do, lets wrap our fetching into a proper script. So that we dont overwhelm their servers, we will only fetch 5 from each page, but you get the idea...

We'll segue of a bit to explore new style format strings. See https://pyformat.info for more info.

"list{:0>2}.txt".format(3)
a = "4"
b = 4
class Four:
    def __str__(self):
        return "Fourteen"
c=Four()
"The hazy cat jumped over the {} and {} and {}".format(a, b, c)
4. Set up a pipeline for fetching and parsing
Ok lets get back to the fetching...

fetched=[]
for i in range(1,3):
    # with open("files/list{:0>2}.txt".format(i)) as fd:
    with open("list{:0>2}.txt".format(i)) as fd:

        counter=0
        for bookurl_line in fd:
            if counter > 4:
                break
            bookurl=bookurl_line.strip()
            stuff=requests.get(URLSTART+bookurl)
            filetowrite=bookurl.split('/')[-1]
            # filetowrite="files/"+str(i)+"_"+filetowrite+".html"
            filetowrite=str(i)+"_"+filetowrite+".html"

            print("FTW", filetowrite)
            fd=open(filetowrite,"w")
            fd.write(stuff.text)
            fd.close()
            fetched.append(filetowrite)
            time.sleep(2)
            counter=counter+1
            
print(fetched)
Ok we are off to parse each one of the html pages we fetched.

We have provided the skeleton of the code and the code to parse the year, since it is a bit more complex...see the difference in the screenshots above.

import re
yearre = r'\d{4}'
def get_year(d):
    if d.select_one("nobr.greyText"):
        return d.select_one("nobr.greyText").text.strip().split()[-1][:-1]
    else:
        thetext=d.select("div#details div.row")[1].text.strip()
        rowmatch=re.findall(yearre, thetext)
        if len(rowmatch) > 0:
            rowtext=rowmatch[0].strip()
        else:
            rowtext="NA"
        return rowtext
Exercise
Your job is to fill in the code to get the genres.

def get_genres(d):
    # your code here
    genres=d.select("div.elementList div.left a")
    glist=[]
    for g in genres:
        glist.append(g['href'])
    return glist

listofdicts=[]
for filetoread in fetched:
    print(filetoread)
    td={}
    with open(filetoread) as fd:
        datext = fd.read()
    d=BeautifulSoup(datext, 'html.parser')
    td['title']=d.select_one("meta[property='og:title']")['content']
    # td['isbn']=d.select_one("meta[property='books:isbn']")['content']
    td['booktype']=d.select_one("meta[property='og:type']")['content']
    # td['author']=d.select_one("meta[property='books:author']")['content']
    #td['rating']=d.select_one("span.average").text
    # td['year'] = get_year(d)
    td['file']=filetoread
    glist = get_genres(d)
    td['genres']="|".join(glist)
    listofdicts.append(td)
listofdicts[0]
Finally lets write all this stuff into a csv file which we will use to do analysis.

df = pd.DataFrame.from_records(listofdicts)
df
# df.to_csv("files/meta_utf8_EK.csv", index=False, header=True)
df.to_csv("meta_utf8_EK.csv", index=False, header=True)
