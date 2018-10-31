# scrapy_tutorial
scrapy tutorial Korean Ver.

이 사이트는 (https://doc.scrapy.org/en/1.3/intro/tutorial.html)를 
제멋대로 한글로 수정해놓은 자료입니다.

scrapy version 1.5 설치된 기준으로 문서가 작성되었습니다.
scrapy 설치방법은 웹사이트를 (scrapy.org) 참조하세요.


지금부터 시작.

http://quotes.toscrape.com/ 이라는 웹사이트를 스크랩 할예정입니다.
해당사이트는 유명 작가의 어록을 적어놓은 사이트네요~


예제는 아래 절차로 진행됩니다:

1. 신규 Scrapy 프로젝트 생성합니다.
2. 사이트를 크롤링하고 자료추출하기 위한 Spider를 작성합니다
3. Command Line을 이용해서 데이터를 뽑아냅니다
4. 링크들을 재귀적으로 따라가기 위해 Spider를 변경합니다.
5. Spider의 arguments들을 사용합니다.

-------------------------------------------------------------------------------------------

1. 프로젝트 생성하기

스크래핑을 시작하기 전에, 새로운 Scrpay 프로젝틀 생성합니다.
디렉토리에 들어가서 아래 코드를 실행합니다.

: scrapy startproject tutorial


그러면,  tutorial이라는 디렉토리가 생성되며 아래의 컨텐츠가 생성됩니다.

tutorial/
    scrapy.cfg            # 환경설정 파일

    tutorial/             # 프로젝트 파이썬 모듈, 여기에 코드를 추가합니다
        __init__.py

        items.py          # 프로젝트 아이템을 정희하는 파일

        pipelines.py      # 프로젝트 파이프라인 파일

        settings.py       # 프로젝트 셋팅파일

        spiders/          # spiders를 작성해서 넣어두는 곳
            __init__.py



2. 스파이더를 작성하기

스파이더는 사용자 정의 클래스(Scrapy가 웹사이트에에서 정보를 스크랩할때 사용) 입니다.
생성시 아래 조건을 만족해야 합니다.
1) 필수적으로 scarpy.Spider의 하위 클래스이어야 합니다. 
2) 필수적으로 initial reqest를 정의해야 합니다.
3) 선택적으로 링크를 어떻게 추적할지 그리고 어떻게 다운받은 사이트를 파싱할지 정의합니다.

아래 코드는 우리의 첫 Spider이며, 이름은 quotes_spider.py 라고 짓겠습니다.
위치는 tutorial/spiders 에 두어야 합니다.


import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log('Saved file %s' % filename)


위의 코드에서 보이듯이, scrapy.Spider 의 하위 클래스입니다.
그리고 몇몇 attribute와 method들을 정의 했습니다.



    name: Spider의 아이디입니다.프로젝트 내에서 다른 Spider들과 이름이 겹칠수 없습니다.

    start_requests(): iterable of Requests 를 반환하는 메소드
                     (a list of requests/generator function)Spider가 크롤링을 시작하는 포인트입니다.

    parse(): 응답받은 request들을 다루는 메소드. 주로 스크랩된 데이터를 추출하고, 새로은 URL을 찾음.


3. 만들어진 spider를 실행하기
 
 spider를 실행하기 위해 프로젝트의 상위 디렉토리에 가서 아래 명령을 실행
 
 : scrapy crawl quotes

 이 명령어는 spider의 quotes를 실행합니다.quotes.toscrape.com으로 요청을 보내고,
 아래와 같은 output을 받습니다.

 ... (omitted for brevity)
2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/> (referer: None)
2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-1.html
2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-2.html
2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
...


실행후 디렉토리를 살펴보면 2개의 파일이 생성,
 quotes-1.html and quotes-2.html, 
  -> 지정한 URL을 이용하혀 parse 메소드가 생성한 파일


[어떤일이 일어난 것일가요?]

Scrapy는 Spider의 Start_request메소드의 scrapy.Request 객체를 예약합니다.
각각의 응답을 받으면 Response객체를 인스턴스화 하고 응답을 인수로 전달을 요청과 관련된 콜백 메소드를 호출


[start_requests 메소드의 지름길]

URL에서 scrapy.Request 객체를 생성하는 start_requests () 메소드를 구현하는 대신,
URL 목록을 사용하여 start_urls 클래스 속성을 정의 할 수 있습니다. 
이 목록은 start_requests ()의 기본 구현에서 사용되어 스파이더에 대한 초기 요청을 작성합니다.

import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)


parse () 메소드는 Scrapy에 명시 적으로 지시하지 않았더라도,
해당 URL에 대한 각 요청을 처리하기 위해 호출됩니다. 
이것은 parse ()가 명시 적으로 할당 된 콜백없이 요청에 대해 호출되는 
Scrapy의 기본 콜백 메소드이기 때문에 발생합니다.



[자료를 추출하기]

Scrapy를 이용해서 자료를 추출하기 위한 가장 빠른 방법은
Scrapy Shell의 Selector를 이용하는것입니다.

scrapy shell 'http://quotes.toscrape.com/page/1/'

(윈도우는 쌍따옴표 사용, scrapy shell "http://quotes.toscrape.com/page/1/")



아래와 같은 모습을 볼수 있음

[ ... Scrapy log here ... ]
2016-09-19 12:09:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
[s] Available Scrapy objects:
[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s]   crawler    <scrapy.crawler.Crawler object at 0x7fa91d888c90>
[s]   item       {}
[s]   request    <GET http://quotes.toscrape.com/page/1/>
[s]   response   <200 http://quotes.toscrape.com/page/1/>
[s]   settings   <scrapy.settings.Settings object at 0x7fa91d888c10>
[s]   spider     <DefaultSpider 'default' at 0x7fa91c8af990>
[s] Useful shortcuts:
[s]   shelp()           Shell help (print this help)
[s]   fetch(req_or_url) Fetch request (or URL) and update local objects
[s]   view(response)    View response in a browser
>>>


Shell을 사용, CSS를 통해 요소를 선택할 수 있음.

>>> response.css('title')
[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]


response.css ( 'title')를 실행 한 결과는
 
 XML / HTML 요소를 둘러싼 Selector 객체의 목록을 나타내는 SelectorList라는 목록과 같은 객체
선택 사항을 추출하거나 추출하기 위해 추가 쿼리를 실행할 수 있음

상기에서 텍스트를 추가 추출하기 위해서는

>>> response.css('title::text').extract()
['Quotes to Scrape']


여기서 확인해야 할 사항은

첫째, <title> 요소에서 직접 텍스트 요소 만 선택하기 위해 CSS 쿼리에 텍스트를 추가 한 것입니다. 
      :: text를 지정하지 않으면 태그를 포함한 전체 title 요소를 얻습니다.

>>> response.css('title').extract()
['<title>Quotes to Scrape</title>']

둘째, selectorList의 인스턴스를 다루기 때문에 .extract ()를 호출 한 결과가 리스트라는 것입니다.  
      이 경우처럼 첫 번째 결과 만 원하면 다음을 수행 할 수 있습니다.

>>> response.css('title::text').extract_first()
'Quotes to Scrape'

다른방법)

>>> response.css('title::text')[0].extract()
'Quotes to Scrape'

그러나, extract_first()를 사용하면 IndexError and returns None을 회피 할수 있음.

extract () 및 extract_first () 메서드 외에도 re () 메서드를 사용하여 정규 표현식을 추출 할 수 있습니다.


정규표현식을 사용한 결과

>>> response.css('title::text').re(r'Quotes.*')
['Quotes to Scrape']
>>> response.css('title::text').re(r'Q\w+')
['Quotes']
>>> response.css('title::text').re(r'(\w+) to (\w+)')
['Quotes', 'Scrape']

사용할 적절한 CSS 선택기를 찾으려면 뷰 (응답)를 사용하여 웹 브라우저의 쉘에서 응답 페이지를 여는 것이 유용 할 수 있습니다. 
브라우저 개발자 도구 나 파이어 버그와 같은 확장 기능을 사용할 수 있습니다 

Selector Gadget은 많은 브라우저에서 작동하는 시각적으로 선택된 요소에 대한 CSS 선택기를 빠르게 찾을 수있는 유용한 도구이기도합니다.


[XPath : 간단한 소개]
CSS 외에도 Scrapy 선택기는 XPath 표현식 사용을 지원합니다.

>>> response.xpath('//title')
[<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
>>> response.xpath('//title/text()').extract_first()
'Quotes to Scrape'

경로 표현식은 매우 강력하며 Scrapy Selectors의 기초입니다. 실제로 CSS 선택기는 Xpath 아래로 변환됩니다. 
쉘에서 선택기 오브젝트의 텍스트 표현을 자세히 읽으면 알 수 있습니다.

아마도 CSS 선택기만큼 인기가 없지만 구조를 탐색하는 것 외에도 내용을 볼 수 있기 때문에 XPath 표현식이 더 많은 능력을 제공합니다. 
XPath를 사용하면 다음과 같은 항목을 선택할 수 있습니다. 

"다음 페이지"라는 텍스트가 포함 된 링크를 선택하십시오. 

이는 XPath를 근근이 살아가는 작업에 매우 적합하게 만들고, 
CSS 선택기를 만드는 방법을 이미 알고 있더라도 XPath를 배우는 것이 좋습니다.

여기에서는 XPath에 대해 다루지 않겠지만 여기에서는 Scrapy Selectors로 XPath를 사용하는 방법에 대해 자세히 설명합니다. 
XPath에 대한 자세한 내용은 예제를 통해 XPath를 배우려면 자습서를 사용하는 것이 좋습니다.


[인용구와 작가 추출하기]

이제 웹 페이지에서 인용 부호를 추출하는 코드를 작성하여 Spider를 완성 해 봅시다.

http://quotes.toscrape.com의 각 인용문은 다음과 같은 HTML 요소로 표시됩니다.

<div class="quote">
    <span class="text">“The world as we have created it is a process of our
    thinking. It cannot be changed without changing our thinking.”</span>
    <span>
        by <small class="author">Albert Einstein</small>
        <a href="/author/Albert-Einstein">(about)</a>
    </span>
    <div class="tags">
        Tags:
        <a class="tag" href="/tag/change/page/1/">change</a>
        <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
        <a class="tag" href="/tag/thinking/page/1/">thinking</a>
        <a class="tag" href="/tag/world/page/1/">world</a>
    </div>
</div>


scrapy 쉘을 열어서 원하는 데이터를 추출하는 방법을 알아 봅시다.

$ scrapy shell 'http://quotes.toscrape.com'

우리는 HTML 요소에 대한 셀렉터 목록을 다음과 같이 얻습니다.

>>> response.css("div.quote")

위의 쿼리에서 반환 된 각 선택기를 통해 하위 요소에 대한 추가 쿼리를 실행할 수 있습니다. 
첫 번째 선택자를 변수에 지정하여 CSS 선택기를 특정 인용에 직접 실행할 수 있습니다.

>>> quote = response.css("div.quote")[0]

이제 방금 생성 한 quote 객체를 사용하여 해당 견적에서 제목, 저자 및 태그를 추출해 보겠습니다.

>>> title = quote.css("span.text::text").extract_first()
>>> title
'“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
>>> author = quote.css("small.author::text").extract_first()
>>> author
'Albert Einstein'

태그가 문자열 목록 인 경우 .extract () 메서드를 사용하여 모든 태그를 가져올 수 있습니다.

>>> tags = quote.css("div.tags a.tag::text").extract()
>>> tags
['change', 'deep-thoughts', 'thinking', 'world']

각 비트를 추출하는 방법을 알아 낸 후 모든 인용 부호 요소를 반복하여 파이썬 Dict에 넣을 수 있습니다.

>>> for quote in response.css("div.quote"):
...     text = quote.css("span.text::text").extract_first()
...     author = quote.css("small.author::text").extract_first()
...     tags = quote.css("div.tags a.tag::text").extract()
...     print(dict(text=text, author=author, tags=tags))
{'tags': ['change', 'deep-thoughts', 'thinking', 'world'], 'author': 'Albert Einstein', 'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'}
{'tags': ['abilities', 'choices'], 'author': 'J.K. Rowling', 'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”'}
    ... a few more of these, omitted for brevity
>>>

[우리의 Spider를 이용해서 자료 추출하기]

특정 데이터를 추출하지 않고 전체 HTML 페이지를 로컬 파일에 저장했는데요, 위의 추출로직을 Spider 스크립트에 통합합니다.

Scrapy 스파이더는 일반적으로 페이지에서 추출한 데이터가 포함 된 많은 사전을 생성합니다. 
그렇게하기 위해 콜백에서 yield Python 키워드를 사용합니다 (아래에서 볼 수 있듯이).

import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }


이 스파이더를 실행하면 추출 된 데이터가 로그와 함께 출력됩니다.

2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are than to be loved for what you are not.”'}
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}


[스크랩된 데이터 저장하기]

스크랩 된 데이터를 저장하는 가장 간단한 방법은 피드 내보내기를 사용하는 것입니다. 다음 명령을 사용하십시오.

scrapy crawl quotes -o quotes.json

quotes.json 파일이 생성됩니다.

전통적으로 Scrapy는 내용을 덮어 쓰지 않고 지정된 파일에 추가합니다. 
두 번째 이전에 파일을 제거하지 않고이 명령을 두 번 실행하면 파손 된 JSON 파일로 끝납니다.

JSON Lines와 같은 다른 형식을 사용할 수도 있습니다.

scrapy crawl quotes -o quotes.jl

JSON Lines 형식은 스트림과 유사하기 때문에 유용합니다. 새 레코드를 쉽게 추가 할 수 있습니다. 
두 번 실행하면 JSON과 동일한 문제가 발생하지 않습니다. 

또한 각 레코드가 별도의 줄이므로 메모리에 모든 것을 넣지 않고도 큰 파일을 처리 할 수 ​​있습니다. JQ와 같은 도구가 있으므로 명령 줄에서이 작업을 수행 할 수 있습니다.

소규모 프로젝트 (이 튜토리얼에서와 같이)에서는 충분합니다. 
그러나 스크랩 한 항목으로보다 복잡한 작업을 수행하려면 항목 파이프 라인을 작성할 수 있습니다. 
항목 파이프 라인에 대한 자리 표시 자 파일은 프로젝트가 생성 될 때 tutorial / pipelines.py에 설정되어 있습니다. 
긁힌 항목을 저장하려는 경우 항목 파이프 라인을 구현할 필요는 없습니다.

[링크따라가기]

예를 들어 http://quotes.toscrape.com의 처음 두 페이지에서 물건을 긁어내는 대신 웹 사이트의 모든 페이지에서 인용문을 원한다고 가정 해 봅시다.

이제는 페이지에서 데이터를 추출하는 방법을 알았으므로 페이지의 링크를 따라하는 방법을 살펴 보겠습니다.

우선, 우리가 따라 가고 싶은 페이지에 대한 링크를 추출하는 것입니다. 우리의 페이지를 살펴보면, 다음과 같은 마크 업과 함께 다음 페이지에 대한 링크가 있음을 알 수 있습니다 :

<ul class="pager">
    <li class="next">
        <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
    </li>
</ul>

Shell에서 아래와 같이 추출할수 있습니다.

>>> response.css('li.next a').extract_first()
'<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'


이것은 앵커 요소를 얻지 만, href 속성을 원합니다. 이를 위해 Scrapy는 다음과 같이 속성 컨텐츠를 선택할 수있는 CSS 확장을 지원합니다.

>>> response.css('li.next a::attr(href)').extract_first()
'/page/2/'

이제 spider가 다음 페이지로의 링크를 따라 재귀 적으로 따라 가면서 데이터를 추출하도록 수정되었습니다.

import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)


이제 데이터를 추출한 후 parse () 메서드는 다음 페이지로 연결되는 링크를 찾고, 
urljoin () 메서드를 사용하여 절대 URL 전체를 빌드하고 (링크가 상대적 일 수 있기 때문에) 다음 페이지로 새 요청을 내고, 
콜백으로 등록하여 다음 페이지의 데이터 추출을 처리하고 크롤링을 모든 페이지에 유지합니다.

여기서 볼 수있는 것은 Scrapy의 링크 연결 메커니즘입니다. 
콜백 메소드에서 요청을 내리면 Scrapy는 요청을 전송하도록 예약하고 해당 요청이 완료되면 실행될 콜백 메소드를 등록합니다.

이 기능을 사용하면 정의한 규칙에 따라 링크를 따라가는 복잡한 크롤러를 구축하고 방문하는 페이지에 따라 다른 종류의 데이터를 추출 할 수 있습니다.
이 예에서는 블로그, 포럼 및 페이지 매김이있는 다른 사이트를 크롤링 할 때까지 다음 페이지에 대한 모든 링크를 따라 가면서 일종의 루프를 만듭니다.


[추가 샘플 및 패턴들]

다음은 콜백을 보여주고 링크를 따르는 다른 스파이더입니다. 이번에는 작성자 정보를 긁어 모으기위한 것입니다.

import scrapy


class AuthorSpider(scrapy.Spider):
    name = 'author'

    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        # follow links to author pages
        for href in response.css('.author + a::attr(href)').extract():
            yield scrapy.Request(response.urljoin(href),
                                 callback=self.parse_author)

        # follow pagination links
        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)

    def parse_author(self, response):
        def extract_with_css(query):
            return response.css(query).extract_first().strip()

        yield {
            'name': extract_with_css('h3.author-title::text'),
            'birthdate': extract_with_css('.author-born-date::text'),
            'bio': extract_with_css('.author-description::text'),
        }

이 spider는 메인 페이지에서 시작할 것이며 각각의 parse_author 콜백을 호출하는 저자 페이지에 대한 모든 링크와 이전에 보았던 parse 콜백과의 페이지 매김 링크를 따라갈 것입니다.

parse_author 콜백은 CSS 쿼리에서 데이터를 추출 및 정리하고 작성자 데이터로 파이썬 dict을 생성하는 도우미 함수를 정의합니다.

이 spider가 보여주는 또 다른 흥미로운 점은 같은 저자의 많은 인용문이 있더라도 동일한 저자 페이지를 여러 번 방문하는 것에 대해 걱정할 필요가 없다는 것입니다. 
기본적으로 Scrapy는 중복 된 요청을 이미 방문한 URL로 걸러내어 프로그래밍 실수로 인해 서버를 너무 많이 치는 문제를 방지합니다. 
이것은 DUPEFILTER_CLASS 설정으로 구성 할 수 있습니다.

Scrap을 사용하여 링크 및 콜백을 따르는 메커니즘을 사용하는 방법을 잘 이해하고 있기를 바랍니다.
링크를 따라가는 메커니즘을 활용하는 또 다른 예제 스파이더는 Crawlers를 작성하는 데 사용할 수있는 작은 규칙 엔진을 구현하는 일반 스파이더에 대한 CrawlSpider 클래스를 확인하십시오.

또한 공통 패턴은 트릭을 사용하여 콜백에 추가 데이터를 전달하는 두 페이지 이상의 데이터로 항목을 작성하는 것입니다.

[Spider 인수 사용하기]

명령 줄 인수를 실행할 때 -a 옵션을 사용하여 스파이더에 명령 줄 인수를 제공 할 수 있습니다.

scrapy crawl quotes -o quotes-humor.json -a tag=humor


이러한 인수는 Spider의 __init__ 메소드에 전달되어 기본적으로 스파이더 속성이됩니다.
이 예제에서 태그 인수에 제공된 값은 self.tag를 통해 사용할 수 있습니다. 

이것을 사용하여 스파이더가 특정 태그로 따옴표 만 가져오고 인수를 기반으로 URL을 구성 할 수 있습니다.

import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        url = 'http://quotes.toscrape.com/'
        tag = getattr(self, 'tag', None)
        if tag is not None:
            url = url + 'tag/' + tag
        yield scrapy.Request(url, self.parse)

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, self.parse)



이 거미에게 tag = humor 인수를 전달하면 http://quotes.toscrape.com/tag/humor와 같이 유머 태그의 URL 만 방문한다는 것을 알 수 있습니다.


