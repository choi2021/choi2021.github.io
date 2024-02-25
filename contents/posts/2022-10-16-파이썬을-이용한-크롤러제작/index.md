---
title: '모으잡-파이썬을 이용한 웹크롤러 제작'
date: 2022-10-16
slug: 모으잡-파이썬을-이용한-웹크롤러-제작
tags: [사이드프로젝트, 모으잡]
series: 모으잡
---

# 파이썬을 이용해 웹크롤러 제작하기

웹크롤러는 데이터를 최신 상태로 유지하거나 웹페이지에 있는 원하는 정보를 추출하는 기술로, 일일이 사용자가 웹페이지를 돌아다니면서 정보를 수집하는 것을 대신해서 방문해 원하는 정보들을 수집해 줄 수 있어 자동화에 용이한 기술이다.

웹크롤러를 만드는 다양한 영상이 유튜브에 있지만 노마드코더(https://nomadcoders.co/courses)의 파이썬 무료 강의를 참고해서 만들어보았다.

파이썬을 이용해 우리가 원하는 website의 정보를 가져오려면 두 가지 라이브러리가 필요하다.

1. requests: 우리가 원하는 데이터 url을 요청할 수 있는 라이브러리다.

2. BeautifulSoup: 요청으로 받아온 response의 HTML 요소들을 가공,처리할 수 있게 도와주는 라이브러리다.

## 1. Requests로 원하는 데이터 요청하기

request을 이용하면 내가 먼저 공부했던 GET, POST, PUT, DELETE HTTP 요청을 할 수 있다.

자바스크립트에서는 fetch를, 파이썬에서는 requests를 이용할 수 있구나 이해가 되었다.

```python
from requests import get

base_url ="https://jsonplaceholder.typicode.com/users/1"
response = get(f"{base_url}")
status_code=response.status_code #요청 코드
content=response.content # 바이너리 원문
text=response.text #인코딩된 문자열
json=response.json() #Json 데이터

```

우리가 요청한 게 올바른지에 따라 에러 핸들링이 필요하기 때문에 status_code로 확인 후에 성공했다면 데이터 추출을 위해 response.text로 응답의 문자열을 받아온다.

## 2. BeautifulSoup

BeautifulSoup은 앞서 requests로 받아온 데이터를 가공하는 데 사용되는 라이브러리이다.

받아온 response.text를 이용해서 자체적인 파싱을 한 후에 우리가 원하는 부분들을 찾아갈 수 있다.

이때 사용하는 메소드는 다음과 같다.

1. find와 find_all ( tag , attr ): 원하는 태그를 찾고 attributes로 자세하게 설정이 가능하다.

2. select( css-selector ): css-selector를 이용해서 우리가 원하는 부분을 찾을 수 있다.

위 두가지 메소드가 유용한 이유는 우리가 원하는 부분을 찾고 난 후에 계속해서 검색할 수 있게 파싱된 text를 return해 준다.

Request와 BeautifulSoup 두 가지를 합쳐서 We Work Remotely라는 사이트의 데이터를 받아오는 코드는 다음과 같이 나타낼 수 있다.

```python
from requests import get
from bs4 import BeautifulSoup


base_url = "https://weworkremotely.com/remote-jobs/search?utf8=%E2%9C%93&term="
response = get(f"{base_url}")
if response.status_code != 200:
   print("Can't request website")
else:
   results = []
   soup = BeautifulSoup(response.text, "html.parser")
   jobs = soup.find_all("section", class_="jobs")  # list로 받아
   for job_section in jobs:
     job_posts = job_section.find_all("li")
     job_posts.pop()  # view_all 제거
       for post in job_posts:
         anchors = post.find_all("a")
         anchor = anchors[1]  # 로고 anchor 제거
         link = anchor["href"]
         company, kind, region = anchor.find_all("span",class_="company")
         title = anchor.find("span", class_="title")
         job_data = {
                    "link": f"https://weworkremotely.com{link}",
                    "company": company.string,
                    "location": region.string,
                    "position": title.string
                }
    results.append(job_data)

```

## 3. 셀레니움

웹크롤링은 우리가 원하는 웹사이트의 정보를 받아오고 가공하는 기술이다. 하지만 회사에서는 우리가 원하는 정보가 중요하기 때문에, 또는 회사 서버에 과부하를 줄 수 있기 때문에 막아놓을 수 있다. 그래서 크롤링을 상업적인 목적이나, 해당 회사에게 불이익을 주면 법적 문제를 가질 수도 있기 때문에 조심해야 한다. 이를 위해서 사용하는 방법이 접속한 게 '봇인지 사람인지'를 확인하는 방식이다.

셀레니움은 이때 우리가 봇으로 접속한 게 아닌 것처럼 실제로 브라우저에 접속할 수 있게 도와줄 수 있는 프로그램이다. 또한, 자바스크립트가 동적으로 만든 데이터들을 크롤링이 가능하고, 사이트의 클릭과 같은 이벤트를 줄 수 있는 장점도 가지고 있다.

셀레니움을 사용하려면 추가적인 설정이 필요하다. 웹드라이버라는 기능으로 우리가 원하는 브라우저에 맞게 설정과 옵션에 따라 설정하면 된다. 나는 replit이란 웹 IDE에서 코딩하고 있었기 때문에 다음과 같은 옵션을 추가해서 진행했다.

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

options = Options()
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
browser = webdriver.Chrome(options=options)
```

이렇게 만든 browser를 이용해 requests를 대신해서 사용하면 다음과 같이 코드를 바꿀 수 있다.

```python
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

options = Options()
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
browser = webdriver.Chrome(options=options)



results = []
for page in range(pages):
   base_url = "https://kr.indeed.com/jobs"
   browser.get(f"{base_url}")
   soup = BeautifulSoup(browser.page_source, "html.parser")
   job_list = soup.find("ul", class_="jobsearch-ResultsList")
   jobs = job_list.find_all("li", recursive=False)

   for job in jobs:
       zone = job.find("div", class_="mosaic-zone")
       if zone == None:
           anchor = job.select_one("h2 a")
           title = anchor["aria-label"]
           link = anchor["href"]
           company = job.find("span", class_="companyName")
           location = job.find("div", class_="companyLocation")
           job_data = {
                "link": f"https://kr.indeed.com{link}",
                "company": company.string,
                "location": location.string,
                "position": title
              }
            results.append(job_data)

```

이렇게 가공한 데이터를 나만 사용할 거 라면, 그냥 로컬로 엑셀로 저장되게 하면 되겠지만, 사용자들이 프론트페이지에서 요청을 보내면, 해당 요청에 맞게 받아오는 서비스를 만들고 싶기 때문에, **백엔드 서버가 필요하게 된다.** 이를 위해서 간단한 flask라는 파이썬 프레임워크를 내일 공부하고 적용해 보고자 한다.
