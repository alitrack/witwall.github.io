# Convert Html to Jupyter Notebook


with [nbconvert](https://nbconvert.readthedocs.io/en/latest/index.html), we can convert Jupyter Notebook(.ipynb) to HTML

with this mehod, we can convert the generated HTML to Jupyter Notebook again.


### method

```python
import requests
from  bs4 import BeautifulSoup
import re
import nbformat as nbf
import os

def html2nb(url,fname=None):
"""
convert Html (converted from Jupyter Notebook) to Jupyter Notebook again.
"""
    if not fname:
        from urllib.parse import urlparse,unquote
        a = urlparse(unquote(url))
        # Get the filename only from the initial file path.
        filename = os.path.basename(a.path)
        # Use splitext() to get filename and extension separately.
        (fname, ext) = os.path.splitext(filename)
        fname = f"{fname}.ipynb"
    res=requests.get(url)
    soup=BeautifulSoup(res.text)
    divs = soup.findAll(attrs={'class': re.compile(r"^cell border-box-sizing .* rendered$")})
    nb = nbf.v4.new_notebook()

    for div in divs:
        if 'text_cell' in div.attrs['class']:
            div=div.find('div',{'class':'text_cell_render border-box-sizing rendered_html'})
            for s in div.select('a.anchor-link'):
                s.extract()
            md = markdownify.markdownify(div.decode_contents())
            cell =nbf.v4.new_markdown_cell(md)
            nb['cells'].append(cell)
        elif 'code_cell' in div.attrs['class']:
            div=div.find('div',{'class':'input_area'})
            code = nbf.v4.new_code_cell(div.text)
            nb['cells'].append(code)
        else:
            pass
    with open(fname, 'w') as f:
        nbf.write(nb, f)
```


### usage

```python
url = "https://slundberg.github.io/shap/notebooks/NHANES%20I%20Survival%20Model.html"
html2nb(url)
```
