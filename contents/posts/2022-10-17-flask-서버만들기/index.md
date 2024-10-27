---
title: "모으잡-flask이용해 SSR서버만들기"
date: 2022-10-17
slug: 모으잡-flask-SSR서버만들기
tags: [사이드프로젝트, 모으잡]
series: 모으잡
---

# Flask를 이용한 서버 제작

flask는 파이썬의 프레임워크로 아주 간단한 방법으로 서버를 구축할 수 있다. flask를 사용하는 방법은 엄청나게 쉬웠는데 route를 연결하고, route에 해당하는 페이지를 간단하게 함수를 이용해서 render_template로 return해주면 해당 페이지를 만들 수 있었다. render_template를 이용할 때 주의할 점은 templates라는 폴더 내에 있는 파일을 전달해주기 때문에 반드시 templates안에 파일을 만들고, 해당하는 파일의 이름을 전달해주면 된다.

```python
from flask import Flask, render_template

app = Flask("JobScrapper")

@app.route("/")
def home():
    return render_template("home.html")

app.run("0.0.0.0") #app을 실행할때 replit은 0.0.0.0을 사용해

```

제작한 스크롤러에 접속할 때 필요한 페이지는 총 세가지로 검색을 하는 "/", 검색한 내용을 보여주는 "/search", 모아온 데이터를 다운받을 수 있는 "/extract"이다.

## home 페이지

홈페이지에서는 검색한 내용을 전달 받아야한다. 전달 받기 위해서 home.html에 form과 input을 이용했다.

form의 action을 이용해서 어디로 보낼지를 정할 수 있고, input의 name은 전달된 url에 query로 전달이 가능하다.

html을 잘 알고 있었다고 생각했지만 form은 항상 event.preventDefault()로 전달을 막고 SPA를 만들다보니 태그 자체가 할 수 있는 기능을 처음 알게 되었다.

```html
<body>
  <main class="container">
    <h1>JonScrapper</h1>
    <h4>What job do you want?</h4>
    <form action="/search">
      <input type="text" name="keyword" placeholder="Write keyword please" />
      <button>Search</button>
    </form>
  </main>
</body>
```

## Search 페이지

home 페이지를 통해 전달한 내용은 "/search/keyword="에 전달되기 때문에 search페이지의 함수에서 keyword를 받아온다.

받아온 keyword를 앞서 만든 크롤러에 전달해 크롤링을 진행한 후에 데이터를 search페이지에 전달해 보여준다.

keyword는 flask에서 request로 url의 해당 부분을 가져올 수 있다. 이때 input에 입력하지 않고 페이지 이동하게 되지 않게

redirect를 이용해 다시 홈페이지로 보내주는 로직도 추가하고, 이후 export에서도 파일을 사용할 수 있게 db라는 딕셔너리에 저장한다.

```python
#main.py
@app.route("/search")
def search():
  keyword = request.args.get("keyword")
  if keyword==None:
    return redirect("/")
  indeed = extract_indeed_jobs(keyword)
  wwr = extract_wwwrjobs(keyword)
  jobs = wwr + indeed
  db[keyword] = jobs
  return render_template("search.html", keyword=keyword, jobs=jobs)
```

```html
<main class="container">
  <h1>Searh Results for "{{keyword}}":</h1>
  <a target="_blank" href="/export?keyword={{keyword}}">Export to file</a>
  <!--extract 페이지로 연결-->
  <table role="grid">
    <hhead>
      <tr>
        <th>Position</th>
        <th>Company</th>
        <th>Location</th>
        <th>Link</th>
      </tr>
    </hhead>
    <tbody>
      {%for job in jobs%} //<!--파이썬 코드-->
      <tr>
        <td>{{job.position}}</td>
        <td>{{job.company}}</td>
        <td>{{job.location}}</td>
        <td>
          <a href={% raw %}"{{job,link}}"{% endraw %} target="_blank"
            >Apply this</a
          >
        </td>
      </tr>
      {%endfor%}
    </tbody>
  </table>
</main>
```

## export페이지

export페이지는 앞서 크롤링한 데이터를 다운로드할 수 있는 페이지다. 똑같이 전달받은 keyword를 이용해서 해당 keyword를 이름으로하는 .csv파일을

만든 후에 클릭시 다운로드를 받을 수 있다. 페이지에서 keyword가 없다면 homepage로 redirect한다. 딕셔너리에 해당 keyword를 키로 하는 내용이 없다면

해당 키워드의 search페이지로 redirect하는 로직을 추가했다. 데이터가 있다면 데이터를 csv파일에 저장후에 flask내장 메소드은 send_file로 다운로드 받을 수

있게 보내준다.

```python
@app.route("/export")
def export():
  keyword = request.args.get("keyword")
  if keyword==None:
    return redirect("/")
  if keyword not in db:
    return redirect(f"/search?keyword={keyword}")
  save_to_file(keyword, db[keyword])
  return send_file(f"{keyword}.csv",as_attachment=True)

```

이렇게 만든 페이지들은 내가 생각했던 프론트와 서버가 구분되는 게 아니라 서버에서 직접 화면을 만들어 보여주는 "서버사이드 렌더링"이라는 것을 공부하고 난 뒤에 깨달았다. 내가 원하는 것은 다음과 같은 과정이었다.

1. API 서버를 만든다

2. 프론트에서 서버에 해당 내용으로 요청을 보낸다
3. 서버는 크롤링을 해서 결과를 json으로 프론트 서버에 응답한다
4. 프론트서버는 받은 내용을 이용해 랜더링한다

파이썬으로는 코딩테스트 공부만했다보니까 실제로 사용하기 위한 환경세팅 자체가 어색했고, flask와 react를 같이 쓰는 자료가 잘나오지 않아서, 이참에 node js를 공부해서 자바스크립트로 프론트와 백엔드를 다 연결해보자라는 생각이 들어, 조금 돌아가지만 크롤링에 대해 이해했으므로, node js를 공부해서 내일 부터 다시 node js를 공부해보기로 계획을 잡았다.
