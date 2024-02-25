---
title: '모으잡-express cors 에러, 에러핸들링'
slug: 모으잡-express cors 에러, 에러핸들링
tags: [사이드프로젝트, 모으잡]
series: 모으잡
date: 2022-10-20
---

# CORS란

CORS는 Cross-Origin Resource Sharing의 약자로, 서버가 알고있는 클리언트의 url에서 요청시에 응답을 막을 수 있는 보안 기능이다. 같은 도메인끼리 밑의 예로 https://localhost:3000이란 주소로 서버와 클라이언트가 작동하고 있다면 둘은 아무 문제 없이 요청과 응답을 한다. 하지만 도메인이 달라질 경우에 예로 서버는 https://localhost:5000에서 클라이언트는 https://localhost:3000에서 작동할 경우에는 다음과 같은 에러를 보여준다.

![img](https://miro.medium.com/max/875/1*XxzATAY3-XDUvB2GJL2QnA.png)

도메인이 같다는 의미는 scheme(https://), hostname(localhost), portNumber(:5000)이 모두 같을 때를 의미한다. 그렇기 때문에 위의 예는 포트넘버가 다르기 때문에 발생한 cors에러라고 할 수 있다.

우리가 원하는 도메인의 클라이언트의 요청만 받기위해서는 서버의 헤더에 **Access-Control-Allow-Origin**과 **Access-Control-Allow-Methods**를 추가해 응답을 보내주면 에러없이 응답을 보내줄 수 있다.

[출처: [medium how cors works](https://medium.com/swlh/how-cors-cross-origin-resource-sharing-works-79f959a84f0e)]

<img src="https://miro.medium.com/max/875/0*SweGXbcps8xY31ds.png" width="700px" />

## Express의 CORS 패키지

저번에 알아봤던 cors 패키지는 위의 두가지 header의 내용을 간단하게 추가할 수 있게 도와준다.

```javascript
app.use(cors({
    origin:["https://localhost:3000"]
    optionSuccessStatus:200
}))
```

origin에 추가한 주소를 통해서 어떤 url의 요청에 추가된 헤더 내용을 담아줄 건지 정해줄 수 있다.

# 에러핸들링

에러핸들링은 크게 동기적인 코드내의 에러와 비동기적인 코드내의 에러로 나눌 수 있다. 기본적으로 에러처리는 발생한 미들웨어에서 최대한 적절하게 해주는 게 중요하다. 혹시 모를 에러를 위해 가장 마지막에 에러처리를 위한 미들웨어를 두어, 서버에러를 알려주는 게 기본적인 구조이다.

### 1. 동기적 코드

동기적 코드는 순서대로 반드시 실행이 마치고 다음으로 넘어가기 때문에 에러 발생시 어플리케이션이 아예 죽어버릴 수 있다. 이를 해결할 수 있는 방법은 try-catch문으로 에러를 잡아서 에러처리를 해준다.

```javascript
app.get('/sync', (req, res, next) => {
  try {
    const data = fs.readFileSync('text.txt'); //동기적으로 파일을 읽어
  } catch (e) {
    res.status(404);
  }
});

app.use((error, req, res, next) => {
  console.error(error);
  res.status(500).json({ message: 'Server error' });
});
```

### 2. 비동기적 코드

비동기적 코드는 try-catch문을 사용할 수 없기 때문에 내부적으로 콜백함수를 이용하거나, catch를 통해 에러를 처리해야한다. 에러를 다음 미들웨어로 넘겨서 처리하고 싶을때는 next에 error를 담아서 보낼 수 있다.

#### 콜백함수

콜백함수로 에러를 처리하는 경우에 에러를 처리해주지 않으면 에러가 났지만 콜백으로 넘어가 클라이언트 페이지는 계속해서 기다리고 있고, 마지막 에러처리 미들웨어까지 전달되지 않으므로, 콜백함수 내에서 처리를 해줘야한다.

```javascript
app.get('/async', (req, res, next) => {
  fs.readFile('/text.txt', (err, data) => {
    if (err) {
      res.status(404).send('File not found');
    }
  });
});

app.use((error, req, res, next) => {
  console.error(error);
  res.status(500).json({ message: 'Server error' });
});
```

### Promise

Promise는 catch를 이용해서 에러 핸들링을 할 수 있다. 에러처리를 해주지 않으면 위와 같이 에러가 마지막 미들웨어로 넘어가지 않는다. 그렇기 때문에 꼭 catch를 이용해서 에러 처리가 필요하다. 밑 예제는 next(error)를 이용해, 마지막 미들웨어로 에러를 전달해주었다.

```javascript
app.get("/async",(req,res,next)=>{
   return fsAsync.readFile('/text.txt').catch((e) => next(e));
  });

})

app.use((error, req, res, next) => {
  console.error(error);
  res.status(500).json({ message: 'Server error' }); //여기서 에러처리
});
```

### async-await

async-await은 내부에서 동기적으로 코드를 사용할 수 있어서 try-catch문을 사용할 수 있고, 함수 자체는 promise로 반환되기 때문에 동일하게 에러처리를 해주지 않으면 에러가 전달되어지지 않는다. 그렇기 때문에 동기적인 코드때 처럼 꼭 try-catch문으로 에러처리를 해주어야한다.

```javascript
app.get('/file3', async (req, res) => {
  try {
    const data = await fsAsync.readFile('/text.txt');
  } catch (e) {
    res.status(404).send('File not found');
  }
});

app.use((error, req, res, next) => {
  console.error(error);
  res.status(500).json({ message: 'Server error' }); //여기서 에러처리
});
```

비동기는 에러가 자동으로 넘어가지지 않는 점 때문에 해당 로직을 항상 처리를 해줘야한다.

마지막으로 에러를 전달할 수 있게 도와주는 패키지로 **express-async-error**가 있어 import만 하면 비동기에서도 에러 발생시 자동으로 마지막 미들웨어까지 전달될 수 있다. 자바스크립트의 비동기 처리와 동일하게 에러처리가 필요하다는 것을 느낄 수 있었다.
