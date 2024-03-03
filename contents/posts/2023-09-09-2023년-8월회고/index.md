---
title: '😊 2023년 8월 회고'
slug: 2023년-8월-회고
date: 2023-09-09
tags: [회고, 리액트네이티브, 에러핸들링, 에러 모니터링]
---

항상 시간은 빠르게 흘러 개발자로 일한지 벌써 6개월이란 시간이 지나버렸다. 이번 한달은 스쿼드의 인원변동이 있어서 업무보다는 모바일 챕터 업무에 많은 시간을 쏟았다. 7월 회고에 언급했던 **예외/에러 처리** 고도화 작업과 함께 React native 발표 자료들을 찾아보다 발견한 성능 측정 툴인 **Flash Light**에 대해서 정리해서 챕터 기술 세션에 발표하기도 했다. 두가지 주제에 대해서 조사하고 공부했던 내용을 적으면서 내가 어떤 점이 부족했는지, 어떤 점은 잘했다고 생각하는지 돌아보려 한다.



### 🥹 본격적으로 시작한 예외/에러 처리 작업

에러/예외 처리 작업은 우리 챕터의 3분기 큰 일감중 하나로 내가 리드를 맡아서 진행하게 되었다. 이를 위해서는 우선 우리 프로젝트에서 사용하고 있는 에러 리포팅 툴인 **Bugsnag**에 대해 잘 알아야했고, 이와 함께 어떤 방식으로 큰 일감을 작은 스토리와 티켓들로 나눠서 작업을 해갈지 **전체적인 로드맵**을 구성해 나가야 했다.



#### 우리가 사용하는 툴, 버그스낵 알아보기

**버그스낵**은 에러/버그 리포팅을 위한 툴이다. 버그스낵을 통해서 어느 코드에서 해당 에러 로그가 제보되었는지 또는 사용자가 호출했던 API나 방문한 페이지 등에 대한 정보들을 기록해준다. 기록된 에러로그는 아래 사진의 대시보드와 같이 모아서 확인할 수 있고 각각을 태그나 에러 메시지 등을 모아서 필터링 하는 등 다양하게 활용이 가능하다.

<br/>

[버그스낵 대시보드 화면]

![image-20230909145825228](https://www.bugsnag.com/wp-content/uploads/2023/05/607f4f6df411bd87c37dd774_prod-searchtab-stacktrace-js.png)

<br/>

먼저 기본 세팅을 하는 것에 있어서 자체 CLI를 제공하기 때문에 DEV/PROD로 구분된 로그 모니터링 프로젝트의 API Key를 연결하고 앱의 source map을 올림으로써 암호화된 JS Bundle 중 어디서 발생된 로그인지를 파악할 수 있다.

[설정 명령어]

```bash
npm install --global @bugsnag/react-native-cli
# or
yarn global add @bugsnag/react-native-cli

# then
bugsnag-react-native-cli init

```



#### Error Boundary

이렇게 설정된 버그스낵을 사용할 때는 버그스낵 자체적인 **Error Boundary** plugin을 이용할 수 있다. 우리 프로젝트는 Error Boundary를 프로젝트 최상위에서 에러를 리포팅 받을 수 있게 설정되어 있는 상태였다.

[버그 스낵 Error boundary 예시 코드]

```tsx
// Start BugSnag first...
Bugsnag.start({...})

// Create the error boundary...
const ErrorBoundary = Bugsnag.getPlugin('react').createErrorBoundary(React)

const onError = (event) => {
  // callback will only run for errors caught by boundary
}

const ErrorView = ({ clearError }) =>
  <View>
    <Text>Inform users of an error in the component tree.
    Use clearError to reset ErrorBoundary state and re-render child tree.</Text>
    <Button onPress={clearError} title="Reset" />
  </View>

const App = () => {
  // Your main App component
}

export default () =>
  <ErrorBoundary FallbackComponent={ErrorView} onError={onError}>
    <App />
  </ErrorBoundary>

```



우리 프로젝트에서 ErrorBoundary가 프로젝트 최상위에 설정되어 있어 핸들링 하지 않는 에러들을 제보받아 로그로 남기고 있었다. 하지만 스크린단이나 컴포넌트 수준에서는 API 호출에 따른 에러를 예외처리를 적용하는 Error boundary가 적용되어 있지 않고 성공/실패/로딩 케이스에 따른 분기문으로 처리된 코드가 일부 남아있었다.  



[성공/실패에 따라 렌더링되는 스크린 코드 예시]

```tsx
const ExampleScreen:React.FC=()=> {
    const [list, setList] = useState([]);
    const [isLoading, setIsLoading] = useState(false);
    const [error, setError] = useState(null);

    useEffect(() => {
        (async () => {
            try {
                setIsLoading(true);
                const result = await getRemoteData();
                setList(result);
            } catch (e) {
                setError(e)
            } finally {
                setIsLoading(false)
            }
        })()
    }, [])


    if (isLoading) {
        return <LoadingView/>
    }

    if (isNotNil(error)) {
        return <ErrorView/>
    }

    return (
        <SuccessView/>
    );
}
```



 이렇게 분기문으로 정의 되어있다면 성공/실패/로딩 세가지에 대한 **관심사가 한곳에 뭉쳐있게** 되기에 이를 해결할기 위해서 성공케이스만 남기고 실패시 보여줘야할 부분은 ErrorBoundary와 Fallback 컴포넌트로 위임시켜 **관심사를 분리**할 수 있어 보였다. 



#### Notify/LeaveBreadCrumb

 버그스낵은  직접 필요한 곳에서 로그를 남길 수 있는 **notify** 메소드를 제공한다.  notify 메소드를 이용하게 되면 원하는 곳에 다양한 데이터를 담아서 보낼 수 있게 되는데 여기에 스크린/파일 위치/ 태그 등을 기록함으로써 보다 로그를 잘 분류하고 한눈에 잘 볼 수 있다. Notify 함수 내부에 event 객체를 받는데 이때 전달받은 event객체의 metadata등 다양한 설정을 추가할 수 있다. 

```typescript
try {
  something.risky()
} catch (e) {
  Bugsnag.notify(error,(event)=>{
    event.severity='warning'; //위험도 설정
    event.unhandled='unhandled'; // unhandled/handled 설정
    event.addMetadata('metadata',{...}); // metadata 추가
    return true      // true로 함으로써 발송해                            
  })
}

```

 

**😅 사용시 주의점 **

notify를 이용해서 버그를 찍어보려할 때 주의할 점은 버그스낵은 **remote Debug tool과 함께 사용하게 되면 정상적으로 로그가 남지 않는다**는 점이다. 이런 이유로는 remote debugger를 사용하게 되면 사용하는 JS 엔진이 달라지기 때문에 정상적으로 버그스낵이 시작되지 못하는 것으로 보인다.

[React native Debugger와 함께 사용했을 때 Warning]

<img width="901" alt="image-20230910011831924" src="https://github.com/choi2021/choi2021.github.io/assets/80830981/4702a0bc-1eba-403b-b716-7f07acf9552f"/>



버그 스낵에는 *breadcrumbs*이라는 짧은 로그가 있어 다음과 같은 기록을 각 에러 로그에 자동으로 남김으로써 디버깅을 도와준다.

- Low memory warnings
- Device rotation
- Menu presentation
- Screenshot capture (not the screenshot itself)
- Undo and redo
- Table view selection
- Window visibility changes
- Non-fatal errors
- Log messages
- User navigation (when enabled through our [navigation plugins](https://docs.bugsnag.com/platforms/react-native/react-native/navigation-libraries/))
- HTTP requests (from JavaScript layer)



이러한 사용자의 동작을 추적함으로써 실제로 불필요한 API가 호출되거나 실패하거나, 메모리 부족으로 발생하는 에러 등을 확인할 수 있었다.

[Breadcrumbs가 남겨진 모습]

<img src="https://docs.bugsnag.com/assets/images/platforms/cocoa-network-crumb.png" width="600"/>

#### Severity

버그스낵에서 에러의 심각도는 2가지로 기준으로 구분된다.

1. handled/unhandled
2. Error/Warning/ Info

조합해보면 총 6가지의 에러 심각도가 나올 수 있지만 unhandled Info와 unhandled Warning을 불필요해보여 아래 **4가지**로 구분하기로 했다.  

- Unhandled Error
- Handled Error
- Warning
- Info

버그스낵 자체 default로는 handled Error는 warning, unhandled Error는 error 심각도로 로그가 남겨진다.



기존 우리 프로젝트 코드는 notify 메소드를 이용해 로그를 찍을 때, 별도의 severity를 설정해서 찍지 않아 기본적인 default severity로 찍히고 있었다. severity를 설정하는 코드는 앞서 설명한 notify 메소드 내부의 onError를 통한 설정으로 예외처리가 되어있는 에러들은 info나 warning으로만 남길 수 있게 적용하려 했다.



#### 이제 일감을 쪼개보자

이렇게 버그스낵를 잘 사용하는 방법에 대해서 조사한 이후에는 다음과 같이 크게 두가지 스텝으로 나누어서 일감을 분리해보았다. 단순히 3분기 내에 모든 작업을 완료할 수는 없을 것이라 생각하고 3분기에는 Step1에만 집중하고자 했다.

<br/>

| **Step 1. 기존 에러 리포트 방식 개선**         | **Step 2. 에러 핸들링 컨벤션 세우기**         |
| :--------------------------------------------- | :-------------------------------------------- |
| 에러 로그 호출 방식 통일화 (완료)              | 에러 바운더리 적용                            |
| 에러 로그 리포트 함수 인터페이스 정리 (진행중) | 비어있는 try-catch 문 제한 커스텀 룰 적용하기 |
| 기존 에러 로그 분류 (진행중)                   |                                               |
| 에러 심각도에 따라 슬랙채널에 공지받기         |                                               |

<br/>

 Step 1은 기존 찍고 있는 로그를 정리하는 작업으로 **의미있고 풍부한 로그를 쌓게하는 작업**과 **로그를 편하게 사용할 수 있게 인터페이스**를 정리하는 작업을 우선으로 잡았다. 그중 8월 한달 동안은 호출 방식을 통일화 하는 작업을 완료했고 인터페이스는 더 필요한 정보들을 잘 전달하기 위해 실제 쌓여있는 로그들을 분석하면서 필수적인 요소들을 담을 수 있게 인터페이스를 정비했다.

9월 동안에는 계속해서 쌓이고 있는 로그들을 분석하면서 변경된 인터페이스에 맞게 더 의미있는 로그들로 바꿔갈 예정이다. 임시적이지만 분류를 위한 기준은 다음과 같이 잡았다.

- Error: 긴급 대응이 필요한 에러를 기록하기 위한 로그
  - 예시: 화이트 스크린, unhandled Error
- Warning: 앱이 죽는 에러는 아니지만 유저 경험에 좋지 않은 상황을 기록하기 위한 로그
  - 예시: Authorization 에러, TIme Out 에러
- Info: 예외 처리가 가능 하지만 기록을 남기기 위한 로그
  - 예시: 요청서/ 견적서 / 결제 등 주요 도메인에 추적을 위한 로그



이 기준은 더 다양한 케이스들을 보고 챕터원들과 의논을 통해 계속해서 확장해 나갈 예정이다. 

 버그 스낵에 있는 수많은 로그들을 보면서 압도되기도 했지만 하나하나 보면서 우리 앱에서 놓치고 있는 부분이나 빠져있는 부분들을 보기도 하고, **Native 에러**를 보면서 모바일 개발자로써 더 많이 공부하고 싶다는 의욕도 생기는 계기가 되었다. 더 튼튼하고 사용성 좋은 앱으로 만들어가는 데 많은 기여를 할 수 있는 일감이 되기를 바라면서 나 또한 더 성장하는 계기가 되어가고 있다.



### 🚀모바일의 Light House, Flash Light

Expo와 React Native 컨프런스인 App.js 채널의 자료들을 찾아보다 앱의 성능 측정에 대한 발표를 보게 되었다. 우리 챕터의 큰 목표중 하나인 성능 개선을 위해서 적용해볼 수 있지 않을까 하는 마음으로 정리했고, 챕터 기술 논의 시간에 발표 자료를 정리해서 전달했다.

본 영상은 웹에서의 성능 측정 도구인 Light house와 같이 객관적으로 측정할 수 있는 툴인 Flash Light에 대한 내용으로 성능 측정을 위한 툴로써 사용방법과 어떻게 사용성을 높여왔는지에 대한 내용이 담겨있었다.

<br/>

**공식 사이트와 관련 자료**

[FlashLIght 발표 영상 ](https://www.youtube.com/watch?v=XO8O1iL4Rxg&t=578s)  

[FlashLight Github Repo](https://github.com/bamlab/flashlight)

[FlashLight 공식 문서](https://docs.flashlight.dev/)

<br/>

Flash Light는 앱 성능 측정 도구로 **안드로이드를** 기준으로 성능을 측정한다. 이러한 기준을 안드로이드로 잡은 이유는 현재 가장 많은 사람들이 사용하고 있는 안드로이드 기기인 삼성 A21과 아이폰 13을 비교했을 때 다음과 같은 차이가 존재하기 때문에 성능이 낮은 안드로이드를 기준으로 잡았다.

<img src="https://github.com/choi2021/choi2021.github.io/assets/80830981/67635a74-a320-48c2-b49a-7ba5edbd7642"/>



발표에서 Flash light를 쓸 수 있는 방법을 5가지로 소개하고 있다. 각 방법은 사용성을 더 높이고 객관적인 데이터를 만들기 위한 방향으로 개선되었다.



#### 1. Flipper Plugin

**Flipper**는 IOS/AOS/RN에서 사용할 수 있는 디버깅 툴로써 디버그 앱을 이용해 성능을 측정하는 방식으로 개발되었다. 하지만 Dev 모드 앱만 측정할 수 있는 한계가 있기 때문에 다음 방식인 명령어를 이용한 방식이 고안되고나서 플러그인은 deprecated 되었다.



#### 2. CLI를 이용한 실시간 측정

다음 방식으로 **CLI 명령어**를 이용한 실시간으로 성능을 측정하는 방법으로 간단한 명령어로 Dev모드 뿐아니라 스토어로 시뮬레이터에 설치한 후에 PROD 모드 앱까지 측정할 수 있는 방법을 소개했다. 사용하는 방법은 다음과 같다

```bash
curl https://get.flashlight.dev | bash  // 명령어 설치
flashlight measure
```



아래 영상을 보면 명령어를 실행하게 되면 localHost로 웹이 켜지는데 자동으로 안드로이드 에뮬레이터를 감지한 후에 해당 에뮬레이터에서 실행중인 앱의 성능을 측정할 수 있다

<br/>

[공식 사이트 CLI 소개 영상]

<iframe width="600" height="450" src="https://github.com/bamlab/flashlight/assets/4534323/4038a342-f145-4c3b-8cde-17949bf52612" frameborder="0" allowfullscreen></iframe>

<br/>

#### 3. 자동화와 유의미한 데이터를 위한 E2E 테스트

 위의 방식으로 실시간으로 측정을 할 수 있지만 항상 우리가 수동으로 측정하기 보다는 **E2E를 이용한 자동화**를 이용해 편의성을 높이려 했다. 공식문서에서 E2E툴로써 [Maestro](https://github.com/mobile-dev-inc/maestro)를 이용한 방식을 추천하고 있다. 자동화 명령어를 이용하면 E2E 코드를 이용해 에뮬레이터에서 자동으로 10번 반복해서 실행하고 데이터를 얻을 수 있다.

 

##### 공식 문서의 트위터를 Maestro를 이용한 E2E 테스트를 측정하는 예시

1.E2E 테스트를 위한 yaml 파일 코드 작성

```
appId: com.twitter.android
---
- launchApp
- assertVisible: Search and Explore
```

2.자동화 명령어

```
flashlight test --bundleId com.twitter.android \ // 실행할 앱의 bundle Id
  --testCommand "maestro test twitter.yaml" \ // 실행할 e2e yaml 파일
  --duration 10000 \ // 측정 시간
  --resultsFilePath results.json // 결과 보고서를 저장하는 방식
```

** 앱의 bundle Id는 `npx @perf-profiler/profiler getCurrentApp` 를 통해서 구할 수 있다.

3.결과 보기

```
flashlight report results.json
```



자동화 방식을 보면서 좀 더 객관화된 데이터를 얻을 수 있고 e2e 테스트를 확장함으로써 각 스크린에 대한 성능을 더 잘 측정할 수 있지 않을까란 기대감이 생겼다.



#### 4. CI와 연동

위의 자동화 script를 AWS Device Farm의 에뮬레이터를 가상 환경과의  연결 기능을 소개했다. 이를 통해서 프로젝트의 CI과정에서 성능 측정을 자동화 함으로써 주기적으로 리포팅을 받을 수 있지 않을까 기대감도 생겼다.



#### 5. Cloud 환경

마지막 apk를 올리고 원하는 e2e테스트 코드를 작성해서 성능을 가상환경에서 측정해주는 자체 서비스를 소개했다. 

사이트 주소는 다음과 같으며 성능 측정이 끝나게 되면 이메일로 결과를 받을 수 있고 결과를 보면 영상으로 나타난다.

FPS, RAM/CPU 사용량 등이 그래프로 나타나지며 영상과 함께 연동되어 있어 어느 지점에서 많은 메모리가 사용되는 지 등을 한눈에 볼 수 있다.

[Cloud 측정 사이트](https://app.flashlight.dev/)



<img src="https://github.com/choi2021/choi2021.github.io/assets/80830981/a727c5ed-1bb8-45ef-bcc5-945929008d4c" width="800"/>



[당시 공유했던 우리 앱의 측정 결과]

<table>
  <tr>
    <th>전체 점수</th>
  </tr>
  <tr>
    <td>
    <img src="https://github.com/choi2021/choi2021.github.io/assets/80830981/e0eea30c-05b9-4c5a-8da4-6c932e6341a2" width="800" />
    </td>
  </tr>
  <tr>
    <th>FPS</th>
  </tr>
  <tr>
    <td><img src="https://github.com/choi2021/choi2021.github.io/assets/80830981/c2a00cf1-52a2-4d48-9a7a-602e105c4cfc" width="800" />
    </td>
  </tr>
  <tr>
    <th>CPU 사용량</th>
  </tr>
  <tr>
    <td><img src="https://github.com/choi2021/choi2021.github.io/assets/80830981/d6231d9e-9712-4dd3-9972-1d03568dadb8" width="800" />
    </td>
  </tr>
  <tr>
    <th>스레드 별 CPU 사용량</th>
  </tr>
  <tr>
    <td><img src="https://github.com/choi2021/choi2021.github.io/assets/80830981/ccb10691-fdcf-4577-b7fe-c442c6e5b707" width="800" />
    </td>
  </tr>
  <tr>
    <th>RAM 사용량</th>
  </tr>
  <tr>
    <td><img src="https://github.com/choi2021/choi2021.github.io/assets/80830981/f0d89756-9649-43f8-bf46-8c0fdcedb066" width="800" />
    </td>
  </tr>
</table>





사양이 아주 낮은 기기를 기준으로 하기 때문에 점수는 낮지만 이를 기준으로 개선되었는지를 보는데 사용할 수 있을 것 같아 보였다. 

발표내용에 대해서 챕터원 분들이 긍정적으로 들어주셨고 성능 개선 업무를 맡은 챕터원 분들께서 직접 좀 더 조사하시고 flash light를 이용해 성능 측정하는 일감에 적극적으로 사용되고 있다. 조사한 내용이 실제로 의미있는 데이터를 모으는데 도움이 되고 있다는 점에 기뻤고 계속해서 기술 세션들에 대해 관심을 갖게 되는 좋은 계기가 되었다.





### 마치며

8월 한달은 챕터업무가 주로 되었던 만큼 기능을 새롭게 개발하는 작업은 적었다. 챕터에 기여하는 작업을 한다는 것은 즐거웠지만 에러와 예외처리를 위해서 기존에 쌓여있던 로그들을 보며 앱이 강제로 종료되거나 화이트스크린이 왜 뜨는지 재현하기 어려워 하루 종일 로그들만 보다가 지나가기도 하고, 필요한 로그를 분류하기보다 불필요하게 쌓이고 있는 로그들을 먼저 제거하게 되기도 하면서 진행 방향이 중간중간 바뀌기도 했다. 순간 순간 하루를 돌아보면서 `내가 의미있는 작업을 하는 걸까, 내가 잘못하고 있는 것은 아닐까` 고민이 되기도 했지만 챕터원들의 격려에 힘을 받아 진행했던 한달이었다. 

에러/ 예외처리라는 일이 당장 눈에 보이는 성과가 적은 일이지만 내가 하는 작업을 통해서 누군가 겪고 있는 에러들을 막으면서 안정성을 높여 더 많은 사람들이 편하게 사용할 수 있기를 기대하고 더 힘내서 진행해보고자 한다. 그리고 혼자서 끙끙대기 보다 각 도메인을 잘 아시는 분들께 적극적으로 도움을 청함으로써 더 효율적으로 일해 9월까지 3분기를 잘 마무리해보려한다. 
