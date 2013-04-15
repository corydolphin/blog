---
layout: post
title: "Converting Mailman Archives to .mbox"
date: 2013-04-14 21:06
comments: true
categories: code
---
Mailman is perhaps one of the most popular mailing lists in the world, yet searching and dealing with archives is absolutely hideous. Mailman stores every email sent and received in a .mbox file internally, but for most mailing lists, this is not accessible, and users must access archives either by vieiwing individual emails in HTML, or else downloading the tar-zipped archives for each month, which are stored in a weird format which is not documented.

For a recent project, I wanted to make a Mailman list's archive searchable, as it contained a great knowledgebase. While many lists are public, these contained potentially private, sensitive information about my university, thus it is not possible to simply use Google to index and search for us.

After a weekend of reverse engineering a number of different email formats that Mailman seems to store archives as, we now have a workin Mailman archive parser, which can be used easily, creatively named [mailman-downloader](https://github.com/wcdolphin/mailman-downloader). 
The script downloads and decodes the archives, searching and replacing email mimetypes which Mailman scrubs, inserting mimetype boundaries and truncating these boundaries as they overflow. Author's email addresses are reformed from the obfuscated "foo   at   bar  dot com" form.

The heart of the decoder:
{% codeblock decode.py %}
        with open(mboxFileName,'w') as outMBox:
            resp = br.open(url)  # get response object
            lines = resp.read().splitlines()  # we actually need the full file, must buffer into memory. 
            lineNum = 0
            while lineNum < len(lines):
                line = lines[lineNum]
                if line.find("From ") == 0: #There are multiple From fields
                    line = line.replace(" at ", "@")
                elif line.find("From: ") == 0: #There are multiple From fields
                    line = line.replace(" at ", "@")
                elif line.find("Message-ID: ") == 0:
                    messageid_stripped = line[line.find('<')+1:line.rfind('>')]
                    messageid_stripped = messageid_stripped.replace('@','')
                    messageid_stripped = messageid_stripped.replace('.','')
                    messageid_stripped = messageid_stripped[0:55]
                    if lineNum +2 < len(lines) and lines[lineNum+2].find('--') ==0:
                        boundary = lines[lineNum+2][2:] #readahead and find the
                        line = line +"\n" + 'Content-Type: multipart/mixed;boundary="%s"' % boundary
                    else:
                        pass
                elif line.find("-------------- next part --------------") == 0:  # some messages have this crap
                    line = None
                elif line.find("Skipped content of type") == 0:  # some messages have this crap
                    line = None
                if line != None:
                    outMBox.write(line + "\n")
                lineNum +=1
        return overWrote, False
{% endcodeblock %}

In the repository, a simple script to upload the downloaded archives to gmail is included, fully automating the process of creating a gmail-based full-text archive searcher, more to come on that soon.

Any questions, comments? Please file an issue and or submit a pull request.