---
title: Script to Embed a Pingback URL Into Word Documents
layout: post
---
A small python script that embeds a 1px image into XML-based Word documents (Word 2010+).  When a user downloads the word document and clicks “Enable Editing”, the pingback URL will be requested.

    Usage: pingback.py <input.docx> <url> <output.docx>

[Source](https://gist.github.com/tifkin-/a29fb9b88f029216d192)

```python
import shutil
import sys
import tempfile
import zipfile

if len(sys.argv) != 4:
    print "Usage: pingback.py <input.docx> <url> <output.docx>"
    exit(0)

inputFile = sys.argv[1]
pingbackUrl = sys.argv[2]
outputFile = sys.argv[3]

tempdir = tempfile.mkdtemp()
tempdir2 = tempfile.mkdtemp()

zf = zipfile.ZipFile(inputFile, 'r')

try:
    zf.extractall(tempdir)
except KeyError:
    print 'ERROR:'+KeyError
finally:
    zf.close()

with open(tempdir+"/word/document.xml","r") as f:
    docData = f.read()

with open(tempdir+"/word/_rels/document.xml.rels","r") as f:
    relsData = f.read()

docData = docData.replace('</w:body>','<w:p><w:pict><v:shape style="width:1px;height:1px"><v:imagedata r:id="XYZA"/></v:shape></w:pict></w:p></w:body>')
relsData = relsData.replace('</Relationships>','<Relationship Id="XYZA" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Target="'+pingbackUrl+'" TargetMode="External"/></Relationships>')


with open(tempdir+"/word/document.xml","w") as f:
    f.write(docData)

with open(tempdir+"/word/_rels/document.xml.rels","w") as f:
    f.write(relsData)


shutil.make_archive(tempdir2+"/out", "zip", tempdir)
shutil.move(tempdir2+"/out.zip", outputFile)

shutil.rmtree(tempdir)
shutil.rmtree(tempdir2)
```

