---
id: 126
title: Accessing Amazons Product API using Clojure
date: 2015-05-31T13:46:00+08:00
author: Sam
layout: post
guid: http://samatkinson.com/?p=126
permalink: /accessing-amazons-product-api-using-clojure/
dsq_thread_id:
  - "3808493395"
categories:
  - Uncategorized
---
I recently embarked upon a new project with some friends called <a href="http://www.swrl.co">Swirl (swrl.co).</a> We decided to write it in Clojure; we're all Java developers by trade but wanted to give clojure a go as a bunch of smart people we know keep saying it's amazing.  I've definitely formed some opinions, but that's for another day (or over a beer).

The purpose of swirl is to store your recommendations. Nothing annoys me more when I finish reading a book and I can't remember who recommended it to me in the first place. Now, if I recommend something to someone I do it through swirl, and they can then respond on the site so I know they've actually watched/read/listened to whatever I recommended.

The first port of call was Amazon integration.  Amazon offers an API (<a href="https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html">The Product Advertising API</a>) to search the site for products and get the details back (and wrapped as an affiliate link).  What I quickly learnt was that the API is horrible to work with for 2 reasons.

1) No JSON option.  Anger!!

2) Every call must include a signature, which is an MD5 hash of all the other arguments and is time-sensitive.

For a read only product API the signature seems massively overkill.  This wouldn't be such a big issue if we'd been using Java, but as I was grappling with Clojure syntax for the first time it meant I got stuck for a long time.  There's default implementations for most languages, so I wanted to make my implementation available for anyone else to use.

The code is available on gist at https://gist.github.com/samberic/27439e09ee336cf3dc61.
<script src="https://gist.github.com/samberic/27439e09ee336cf3dc61.js"></script>

The two key methods here are search-book, which uses the API to search through all of amazons catalog to return a series of results, and get-book which retrieves the details of a specific book by ASIN.  The url creation (e.g. the tricky bit) is done in search-url.
<pre class="EnlighterJSRAW" data-enlighter-language="null">(defn search-url [bookname]
  (createEncryptedUrl (assoc (timestamp params) :Keywords bookname :Operation "ItemSearch" :SearchIndex "Books"
                                                )))</pre>
There are a bunch of parameters I've hard coded- amazon key and associate tag, service and version (no need to change these) and ResponseGroup.  ResponseGroup lets you choose what you want returned from your search, and you must have at least one.

I merge this standard list with the current timestamp, the book to be searched for, and the operation and search index.  If you want to search for things other than books, you can simply change the search index. Once we have a full map of parameters we can use this to create the signature.

&nbsp;

The signature is made of an alphabetically-sorted (hence why params is a sorted-map) list of all parameters joined using "&amp;", and properly encoded (this was a real gotcha: I had to do string replace on +/%20 in as neither form-encode or url-encode got the job done properly. Hack!), preceded with "GET<span class="pl-cce">\n</span>webservices.amazon.com<span class="pl-cce">\n</span>/onca/xml<span class="pl-cce">\n", which you then put through a "RFC 2104-compliant HMAC with the SHA256 hash" using your amazon private key.</span>

&nbsp;

After much bashing of head against desk, the amazing <a href="http://danielflower.github.io/">Dan Flower</a> pointed me to <a href="http://funcool.github.io/buddy-sign/latest/">Buddy</a>, a clojure security library we already had in the project anyway which took care of the hashing in a clear and concise way.

I hope this helps someone out somewhere.

<script src="https://gist.github.com/samberic/27439e09ee336cf3dc61.js" type="mce-no/type"></script>