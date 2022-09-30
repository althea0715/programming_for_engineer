# Sphinx

## 기본적으로 설치해야할 것들

> python -m pip install sphinx sphinx-book-theme myst-nb

위 명령어를 통해 테마와 markdown, ipynb를 처리할 수 있습니다. ipynb를 사용하기 위해 pandoc을 설치해주세요.


## conf.py 설정방법

2022년 08월 21일 작성

저는 보통 sphinx 설정시에 docs 폴더에 구성하며 html과 source를 구분하는 것을 선호합니다. 이 상황을 기준으로 conf.py 기록을 남깁니다.


```python
# Configuration file for the Sphinx documentation builder.

import sys
import os

sys.path.insert(0, os.path.abspath("../.."))    # 최상단부터 코드를 검색하기 위해

# -- Project information -----------------------------------------------------

project = "study"
copyright = "2022, althea"
author = "althea"

# -- General configuration ---------------------------------------------------
extensions = [
    "sphinx.ext.napoleon",  # rst말고 md 형태의 데이터도 읽기 위해서 사용합니다.
    "sphinx.ext.mathjax",   # md에서 수식을 입력하기 위해 사용합니다.
    "sphinx.ext.autodoc",   # code를 자동 문서화하기 위해 사용합니다.
    "myst_nb",              # md, ipynb를 읽기위해서 사용합니다.
]

source_suffix = {
    ".rst": "restructuredtext",
    ".ipynb": "myst-nb",
    ".myst": "myst-nb",
}

myst_enable_extensions = ["amsmath", "dollarmath"]  # dollarmath 옵션을 설정해야 $$로 쌓인 수식을 표현할 수 있습니다.

templates_path = ["_templates"]
exclude_patterns = []


# -- Options for HTML output -------------------------------------------------

html_theme = "sphinx_book_theme"    # 최근엔 이 테마가 이쁘더라구요.
html_theme_options = {
    "extra_navbar": "",             # 좌하단 해당 테마 사이트 링크 삭제
    "use_download_button": False,   
    "use_fullscreen_button": False,
}

html_static_path = ["_static"]
html_show_sourcelink = False
```