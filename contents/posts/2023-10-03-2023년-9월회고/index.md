---
title: '🚅 2023년 9월 회고'
slug: 2023년-9월회고
date: 2023-10-03
tags: [회고]
---

9월동안 했던 업무와 학습한 내용들을 돌아보면 스쿼드 업무로는 사내의 새벽점검작업을 참여했고, 우리 제품의 중요한 도메인중 하나인 바로견적에 대한 개선 작업을 했다. 개인적으로는 매일 사용하지만 부족함을 느꼈던 부분들을 공부했다.

### 😅 처음해본 새벽 점검 작업 
견적발송에 관한 책임이 우리 스쿼드로 모두 넘어오면서 추석과 연휴를 앞두고 필요했던 9월 새벽점검 작업을 진행하게 되었다. 점검을 위해서 PO분과 백엔드개발자 분들에 비해 상대적으로 적은 작업을 했지만, 점검을 확인하기 위해 **점검 우회가 가능한 버전**의 앱을 빌드해 공유드리는 역할을 맡았다.

일을 하면서 전 작업자분들과 많이 이야기하고, 스쿼드 내부 점검 테스트도 진행하며 실서버에서 점검을별다른 추가 작업이 없어도 될 것이라 예상했다. 하지만 전달드린 빌드버전이 PROD 서버의 점검을 우회하지 못하는 상황이 벌어졌다. 많이 당황스러웠지만 최대한 빨리 전달드려야할 것 같아 원인을 찾아보았고, 원인은 **점검우회를 위한 키값 설정** 때문이었다.

우리 앱의 환경변수를 설정할 때 **config**를 이용해 서드파티 라이브러리들에 필요한 키값 등을 설정하고 있는데 Dev/Prod에 따라 서버점검 우회 키값이 다르게 설정되어 있다. 내가 전달드린 테스트용 빌드 버전은 dev 버전의 키값이기 때문에 스쿼드 내부 점검 테스트는 우회할 수 있었지만, Prod 서버에 접근할 때는 Prod로 설정되어있는 서버점검 우회 키값이 필요해 발생한 상황이었다.

Prod 키 값으로 임의로 수정한 버전을 다시 빌드해 전달드리면서 해결할 수 있었지만, 서비스에 영향을 주지않기 위해 최대한 빠르게 작업을 진행해야하고 사전에 세워둔 프로세스에서 PO, QA분들이 진행해야할 작업이 늦춰지는 것 같아 많이 당황스러운 순간이었다 😂

사전에 PROD 서버 점검을 위해서는 내부테스터 또는 testflight 앱을 다운받아서 QA 하실 수 있게 안내했어야했는데 사전 준비와 우리 앱의 동작이 어떻게 되는지에 대해 이해가 부족해 생긴 이슈였다... 하지만 개인적으로는 모바일 환경에 대해 더 많이 배울 수 있고 이해도를 높일 수 있는 좋은 기회가 되었고, 이후 점검시 더 편하게 진행할 수 있게 세팅을 수정하는 작업도 추가적으로 진행했다.

 ![image-20231003201510681](https://github.com/choi2021/choi2021.github.io/assets/80830981/d1c3d88a-6da4-475e-b9ea-0616eb100d8c)  



### 😎React-native-reanimated 써보기

스쿼드 작업으로 바로견적 지역설정 개선 일감을 진행하면서 **react-native-reanimated** 라이브러리를 이용했다. 라이브러리를 이용하면서 공부했던 내용을 정리해보려 한다.

React-native-reanimated는 react-native 환경에서 애니메이션을 60FPS에 맞게 구현할 수 있게 도와주는 라이브러리다. 저번달의 sticky 헤더 애니메이션을 구현하면서도 사용하려 했지만, 당시 v1으로 설치되어있었기 때문에 react native 버전을 0.71로 올리는 작업과 병목이 있어 react-native 자체 animated api를 이용했다. 결국은 안드로이드에서... 좌절하게 되었지만...🥲

그러면 왜 reanimated는 자체 React-native의 Animated보다 성능을 잘 보장할 수 있을까? 



##### 짤막한 React-native-reanimated의 원리 

 react-native-reanimated를 이해하기 위해서는 먼저 RN이 두개의 스레드로 이루어져있다는 점을 알아야한다.

RN의 두가지 스레드는 **JS 스레드**와 **UI스레드**로 구성되어있는데 두가지 스레드가 하는 역할이 다르다.

- JS 스레드: 내가 작성하는 JS 코드(react)가 동작하는 곳
- UI 스레드: native(IOS/AOS) 코드가 동작하는 곳 

두가지 다른 스레드는 **Bridge**를 통해서 JSON 형식으로 소통해 내가 원하는 UI를 만들게 된다. 이러한 소통과정에서 JS 스레드에서 시간이 많이 소요되는 작업이 진행중이라면 UI 스레드에 반영이 늦어져 **프레임 드랍**이 발생하게 되면서 우리가 원하는 자연스러운 애니메이션 구현에 어려움이 생기게 된다. 

<br/>

<img src="https://blog.xmartlabs.com/images/powerful-animations-rn/frame_drops.png"/>

저번달에 애니메이션의 어려움을 겪었던 이유를 분석했던 것 처럼 layout을 이동시키는 것은 RN의 Animated API를 이용할 때 useNativeDriver를 사용할 수 없기 때문에, JS 스레드에서 동작하기 때문에 버벅임이 발생한 것으로 이해할 수 있었다.



그러면 reanimated는 이러한 문제를 **어떻게** 해결하는 걸까?

<br/>

reanimated는 자연스러운 애니메이션을 위해 두가지를 이용하는데 바로 **Shared value**와 **worklet** 함수이다. 

sharedValue는 JS스레드와 UI스레드 모두에서 읽을 수 있는 값이고 worklet함수는 UI 스레드에서 작동하는 자바스크립트 함수를 의미하는데, 애니메이션을 비동기적으로 전달하는 것이 아니라 **UI 스레드에서 바로 동작**할 수 있게함으로써 성능을 보장한다. 

```jsx
function App() {
  // shared value
  const sv = useSharedValue(0);

  const handlePress = () => {
    sv.value += 10;
  };
  
  // worklet function
  const style = useAnimatedStyle(() => {
    console.log('Running on the UI thread');
    return { opacity: 0.5 };
	});

}
```



JS 코드를 바로 UI 스레드로 동작하게 위해서는 Reanimated의 자체 babel plugin을 통해 worklet함수를 파악함으로써 동작할 수 있다. 왜 reanimated 설치시 `babel.config.js`에 플러그인을 추가하게 하는지 이해할 수 있었다.



[Reanimated 설치시 필요한 플러그인]

```tsx
module.exports = {
    presets: [
      // ... // don't add it here :)
    ],
    plugins: [
      ...
      'react-native-reanimated/plugin',
    ],
  };
```

좀 더 깊은 내용은 [Reanimated 플러그인 README](https://github.com/software-mansion/react-native-reanimated/blob/main/plugin/README-dev.md)를 통해 이해할 수 있다.

<br/>

[Reanimated를 이용해 UI 스레드에서 동작하는 애니메이션 코드]

<img src="https://blog.xmartlabs.com/images/powerful-animations-rn/declarative_way.png"/>



##### Reaniamted를 이용해 내가 만들었던 애니메이션 

간단하게 reanimated를 통해 구현했던 애니메이션 코드를 정리해보려 한다. 내가 구현해야하는 요구사항은 조건에 따라 롤링이 되는 섹션으로 조건에 따라 다음으로 넘어가듯이 나타나게 되는 애니메이션이었다. 

이를 위한 코드 예시는 간단히 다음과 같다.

```tsx
export default function App() {
    const animatedOpacity = useSharedValue(1);
    const animatedTranslateY = useSharedValue(0);
    const animatedStyle = useAnimatedStyle(() => {
        return {
            opacity: animatedOpacity.value,
            transform: [{ translateY: animatedTranslateY.value }],
        };
    });

    const handlePressSteady = () => {
        animatedTranslateY.value = withTiming(0, { duration: DURATION });
        animatedOpacity.value = withTiming(1, { duration: DURATION });
    };

    const handlePressUp = () => {
        animatedTranslateY.value = withTiming(-DISTANCE, { duration: DURATION });
        animatedOpacity.value = withTiming(0, { duration: DURATION });
    };

    const handlePressDown = () => {
        animatedTranslateY.value = withTiming(DISTANCE, { duration: DURATION });
        animatedOpacity.value = withTiming(0, { duration: DURATION });
    };

    return (
        <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
            <Animated.View style={animatedStyle}>
                <View
                    style={{
                        width: 100,
                        height: 100,
                        backgroundColor: 'teal',
                        justifyContent: 'center',
                        alignItems: 'center',
                    }}>
                    <Text style={{ color: 'white', fontSize: 30 }}>돌거야</Text>
                </View>
            </Animated.View>
            <View style={{ flexDirection: 'row', marginTop: 100 }}>
                <Button onPress={handlePressUp} title='올라가유' />
                <Button onPress={handlePressSteady} title='보여유' />
                <Button onPress={handlePressDown} title='내려가유' />
            </View>
        </View>
    );
}
```

<br/>

위 코드로 구현한 애니메이션은 다음과 같다.

<img src="https://github.com/choi2021/choi2021.github.io/assets/80830981/836938bb-bc34-4189-8bf6-ca7c23c6d11e" width="300"/>



이제 하나일때가 아니라 여러개 일때의 애니메이션을  만들어 보자.

```tsx
const RollingView: React.FC<{ text: string; show: boolean }> = ({ text, show }) => {
    const animatedOpacity = useSharedValue(1);
    const animatedTranslateY = useSharedValue(0);
    const animatedStyle = useAnimatedStyle(() => {
        return {
            opacity: animatedOpacity.value,
            transform: [{ translateY: animatedTranslateY.value }],
        };
    });

    const startAnimation = useCallback(() => {
        if (show) {
            animatedTranslateY.value = withTiming(0, { duration: DURATION });
            animatedOpacity.value = withTiming(1, { duration: DURATION });
        } else {
            animatedTranslateY.value = withSequence(
                withTiming(-DISTANCE, { duration: DURATION }),
                withTiming(DISTANCE, { duration: 0 }),
            );
            animatedOpacity.value = withTiming(0, { duration: DURATION / 2 });
        }
    }, [animatedOpacity, animatedTranslateY, show]);

    useEffect(() => {
        startAnimation();
    }, [startAnimation]);

    return (
        <Animated.View
            style={[
                {
                    backgroundColor: 'teal',
                    position: 'absolute',
                    justifyContent: 'center',
                    alignItems: 'center',
                },
                animatedStyle,
            ]}>
            <Text style={{ color: 'white', fontSize: 30 }}>{text}</Text>
        </Animated.View>
    );
};

export default function App() {
    const [showingNumber, setShowingNumber] = useState(1);

    const handlePressOne = () => {
        setShowingNumber(1);
    };

    const handlePressTwo = () => {
        setShowingNumber(2);
    };

    return (
        <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
            <View
                style={{
                    position: 'relative',
                    justifyContent: 'center',
                    overflow: 'hidden',
                    width: 100,
                    height: 100,
                }}>
                <RollingView text={'1번'} show={showingNumber === 1} />
                <RollingView text={'2번'} show={showingNumber === 2} />
            </View>
            <View style={{ flexDirection: 'row', marginTop: 100 }}>
                <Button onPress={handlePressOne} title='1번 보여줘' />
                <Button onPress={handlePressTwo} title='2번 보여줘' />
            </View>
        </View>
    );
}

```

위  코드를 동작시키면 다음과 같은 애니메이션이 만들어진다.



<img src="https://github.com/choi2021/choi2021.github.io/assets/80830981/0a45c37e-3ac9-4090-a5a9-696d49430c6f" width="300"/>



내가 한 방법이 최선은 아닐 수도 있지만 reanimated를 이용해서 애니메이션 요구사항을 AOS/IOS 모두에서 적절하게 구현할 수 있어서 뿌듯했던 일감이었다.



### 😊하루하루 내가 공부해온 것들

개발자로 8개월이란 시간이 쌓이면서 내가 우선시해서 공부해야할 것이 무엇일까 고민했다. 9월은 블로그에 기록하지는 않았지만 3월부터 클린코드, 클린 아키텍처, 함수형 프로그래밍, 리팩터링 4권의 책을 읽었고 다음은 어떤걸 공부해볼까 고민이 되는 시점이었다. 아침 저녁 출퇴근 시간에 선배 개발자분들의 유튜브 영상들을 많이 찾아보는데, 그때 **드림코딩**의 [개발 공부 제대로 하는 법 🤓 (정체기에서 성장기로 가보자, 함 해보자!)](https://www.youtube.com/watch?v=DmK7d0xB2j0) 의 일정관리 부분에 대해 들으면서 내가 우선해서 학습해야할 내용은 어떤 것인지 정해보게 되었다.



##### 타입스크립트를 아세요? 🧐

나는 **react-native**를 이용해 모바일 엔지니어로 일하고 있고 개발 언어는 **Typescript**로 개발하고 있다. 그렇다면 내가 더 많이 회사에 기여하고 성장하기 위해서 우선시 해서 잘해야할 부분이 어떤 것인지에 대해 생각해보았을 때, 가장 먼저 떠올랐던 부분이 `Typescript` 였다.

"나는 타입스크립트를 잘알고 있나?"라는 질문을 했을 때 그렇지 않다는 생각이 들었다. 단순히 내가 type alias와 interface를 정의하고 사용하는 것에 그치고 있었다. "그러고 보니 나는 한번도 타입스크립트 공식문서를 보지 않았구나... "라는 생각에 **하루에 한페이지**씩 타입스크립트 공식문서 핸드북을 정리해보자는 목표가 생겼다. 

나름 매일 한 챕터씩 정리해서 현재 거의 대부분의 핸드북 내용을 한번씩 다읽어보았다. 읽으면서 왜 코드리뷰때 switch문의 default문을 작성할 때 never 타입으로 변수를 만들어서 적용하는 게 좋겠다고 말씀해주셨는지, enum과 object는 어떻게 다른지 등등을 알 수 있었다.

내맘대로 정리한 Typescript Handbook 내용은 조금 더 다듬어서 typescript 섹션에 정리해보려 한다.



[내맘대로 정리한 Typescript HandBook 내용]

<img width="400" src="https://github.com/choi2021/choi2021.github.io/assets/80830981/ab929707-55dd-49b6-a4b2-1ea115c9806c"/>



##### Native도 공부해보자 🫡

챕터내 업무로 에러/예외처리 고도화 업무를 맡아서 진행하면서 네이티브 에러들을 보면 잘 이해가 안되고 해결하지 못해서 답답하게 느껴졌다. 우리 프로젝트 내의 코드가 아니라 라이브러리들이나 react-native가 가지고 있는 문제로 인해 발생하는 경우가 많았기 때문에  `java`라고 적혀있기만 하면 괜히 무서운 느낌이 들고, 해결하기 어렵겠다는 생각에 후순위로 미뤘다. 

하지만 Firebase의 Crashanalytics에 제보되기도 하고 에러로그에 갑자기 죽는 로그들을 보면서 내가 더 공부한다면 더 많은 것을 우리 챕터에 기여할 수 있겠다는 생각이 들어 Native에 대한 이해도를 해결하고자 **안드로이드**를 공부하기 시작했다.

내가 처음 자바스크립트를 배울 때 처럼 우선은 코틀린 문법, 코딩테스트 문제들을 풀고, 클론 코딩 강의들을 보면서 어떻게 앱을 만들어가는지를 보고 따라하며 학습하고 있다. 학습할 수록 기존에 사용하던 자바스크립트, 파이썬과 비교하면서 `동적 타입 언어와 정적타입 언어`의 차이점들을 많이 느낄 수 있었고, 자바스크립트와 코틀린이 어떤 점이 다른지 어떤 점은 비슷한지 느끼며 해당 부분도 정리해가고 있다. 이부분도 블로그에 추가적으로 기록해나갈 예정이다.



#### 마치며

멀리 멋있어보이는 것들이 나올 때마다 저것을 배우지 않으면 뒤쳐지지 않을까 걱정도 되지만, 지금 내가 있는 자리에서 필요한 부분들을 채워나가는 것을 먼저해야겠다고 다시 한번 생각한 한달이었다. 매일 사용하는 타입스크립트와 Git, 리액트와 네이티브 코드들, 충분히 잘 이해하고 있는지 점검해보면서 우선순위를 세울 수 있었다. 더 많이 나누고 더 많이 기여하기 위해서는 내가 먼저 채워져야함을 많이 느낀다. 아직 8개월 차 응애(?)고 매일매일 새롭게 배울게 넘쳐나지만 여기서 주어지는 키워드들을 놓치지 않고 내것으로 만들다보면 내가 보고 배워야겠다고 느끼는 동료들의 모습을 나도 가지게 되어가지 않을까 생각한다. 그냥 꾸준히 하는 것, 내가 제일 잘하는 **꾸준히**를 매일 해보자.



[참고한 자료]

- [reanimated 공식문서](https://docs.swmansion.com/react-native-reanimated/docs/fundamentals/glossary/#ui-thread)
- [reanimated 플러그인 Readme](https://github.com/software-mansion/react-native-reanimated/blob/main/plugin/README-dev.md)

- [Powerful animations in React Native](https://blog.xmartlabs.com/2020/04/27/powerful-animations-in-RN/)
- [원리와 예제를 통해 React-native-reanimated V2 입문하기](https://medium.com/crossplatformkorea/%EC%9B%90%EB%A6%AC%EC%99%80-%EC%98%88%EC%A0%9C%EB%A5%BC-%ED%86%B5%ED%95%B4-react-native-reanimated-v2-%EC%9E%85%EB%AC%B8%ED%95%98%EA%B8%B0-336e832f6ed6)
