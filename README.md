# Category-Theory-for-Programmers

~~~python
import os
from tqdm import tqdm
from docling.document_converter import DocumentConverter

"""
url = "https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/"
"""

os.makedirs("results", exist_ok = True)
converter = DocumentConverter()
with open("./urls.txt", "r") as f:
    lines = f.readlines()
    
    for line in tqdm(lines):
        doc = converter.convert(line).document
        mk_doc = doc.export_to_markdown()
        mk_doc = mk_doc[:mk_doc.rfind('### Share')]
        mk_doc = mk_doc.replace('\r','')
        mk_doc = mk_doc.replace('&gt;','>')
        mk_doc = mk_doc.replace('&lt;','<')
        mk_doc = mk_doc.replace('&amp;','&')
        line = line[:-2]
        name = line[line.rfind('/') + 1:] + '.md'
        with open(os.getcwd() + '/results/' + name, "w") as ff:
            ff.write(mk_doc)

~~~

* 번역은 gemma3-4b을 이용하였습니다.