---
layout: post
title: "Creating Custom SOAP Requests with Ruby and Net::HTTP"
date: 2008-12-19
comments: true
categories: 
- Ruby
- Rails
---

{% img /images/posts/golf_scale.jpg %}

I interviewed at a company a while ago and one of the questions they asked me was the familiar "You have 8 golf balls and a justice scale. One of the golf balls has a bubble in it making it a tiny bit lighter, but you can't tell by looking at it or holding it. How many times do you need to use the scale in order to find the defective ball?". I left the interview thinking that the answer was 3. After about 3 hours of thinking about it, I realized that the answer was 2 tries.

The reason I share this with you is because today I found the answer to something that I've been thinking about for much longer than 3 hours. For about 1 1/2 years, I have been trying to find a good way to simply post a SOAP request via ruby without having to use SOAP4r or WSDL's. Some of you might find this a bit odd that it took me this long to figure this out, but then again, it's almost impossible to find any examples of this online.

Enough history. Let's get to the code.

In my book, [Rails Pocket Reference](http://www.amazon.com/gp/product/0596520700?ie=UTF8&tag=kobu-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0596520700), I have a section about using SOAP. In the example, I used the soap/wsdlDriver library to parse a WSDL from GeoCoder.us, a free US address geocoder tool. Instead of using the WSDL, you can access the response directly.

```ruby
require 'net/http'
require 'net/https'

# Create te http object
http = Net::HTTP.new('rpc.geocoder.us', 80)
http.use_ssl = false
path = '/service/soap/'

# Create the SOAP Envelope
data = <<-EOF
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
  SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
  xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
  <SOAP-ENV:Body>
    <m:geocode xmlns:m="http://rpc.geocoder.us/Geo/Coder/US/">
      <location xsi:type="xsd:string">1005 Gravenstein Highway North Sebastopol, CA 95472</location>
    </m:geocode>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
EOF

# Set Headers
headers = {
  'Referer' => 'http://www.appfusion.net',
  'Content-Type' => 'text/xml',
  'Host' => 'rpc.geocoder.us'
}

# Post the request
resp, data = http.post(path, data, headers)

# Output the results
puts 'Code = ' + resp.code
puts 'Message = ' + resp.message
resp.each { |key, val| puts key + ' = ' + val }
puts data
```

I hope this makes using SOAP with Ruby a lot less cryptic for other as it does me.
