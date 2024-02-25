---
title: '원티드 프리온보딩 2-2 과제회고'
date: 2022-11-07
slug: wanted-pre-onboarding-2-2-review
tags: [회고, 원티드프리온보딩]
series: 원티드프리온보딩
---

# 📜 과제 설명

이번 과제는 이번 기간에 참가한 기업의 과제는 아니지만 이전 기수에서 주어졌던 과제로, 멘토님께서 좋은 과제라 생각하셔서 넣으셨다고 했다. 기업은 광고 회사로 광고와 마케팅과 관련된 데이터들을 보여 주는 **대시보드와 관리 페이지**를 만드는 과제였다. 난생 처음 보는 용어들과 너무 디테일 하게 구성되어있는 피그마 페이지 때문에 여태 까지 과제들 중에서 가장 힘든 과제가 되었다...

저번과 동일하게 **Typescript**와 **Context API, useReducer**를 활용해 상태 관리를 했다. 그래프를 위해서는 APEXCHART 라이브러리, 날짜 변경을 위해서는 reat-picker 라이브러리를 이용해 진행했다. 이번에 과제를 하면서 시도해보고 싶었던 것은 두 가지로, 수업을 통해 배웠던 **관심사 분리**와 **redux**를 이용한 전역 상태 관리였다. 하지만 기능 구현에도 너무 많은 시간이 들어, redux를 시도할 시간이 없이 빠르게 ContextAPI를 이용해 전역 상태를 관리했다.

## 🎈 관심사 분리

관심사 분리는 좋은 코드를 작성하기 위한 기준이 되어 준다. **KISS (Keep it simple stupid)**라고 불리는 원칙은 하나의 모듈이 하나의 기능만을 하게하라는 뜻이다. 처음에 생각했을 때 하나의 모듈이 여러가지 기능을 하게 되면 더 좋지 않을까 라는 생각을 했지만, 문제가 생겼을 때 여러 기능이 엉켜 있다면 해당 문제를 해결하려다 다른 기능이 영향을 받는 일이 생기게 된다. 이러한 상태를 흔히 "스파게티 코드"라고 불린다.

사실 함수를 배우면서 하나의 기능만 하게 해야 한다는 원칙은 너무 많이 들어왔지만, 나중에 내가 했던 프로젝트의 코드를 돌아보면 도저히 손댈 수 없이 엉켜있는 것을 보게 된다. 이러한 하나의 모듈이 하나의 기능만 하기 위해서는 서로 다른 관심사를 하나에 두지 않는 **관심사 분리**가 필요하다. 내가 좋아하는 자바스크립트는 멀티 패러다임 언어이기 때문에, 함수형 프로그래밍과 객체지향 프로그래밍 두 가지 모두 가능해, **class**로 모듈을 나누고, 필요한 곳에 사용할 수 있게 전달 **(Dependency Injection)**하면 사용하는 곳에서는 어떻게 구성되어있는지 자세히 알지 못해도 사용할 수 있는 **추상화**가 된 모듈들로 연결해 나갈 수 있다.

이전에 class를 사용할 때 하나의 기능을 하는 관련된 것들을 모아두자는 기준으로 index.tsx에서 instance를 만들어 전달하는 방식을 알고는 있었지만 사용하는 곳까지 계속해서 prop으로 전달하는 방식을 이용했다. 이러한 방식은 똑같이 prop-drilling이 발생하게 되고 오히려 더 복잡하게 되는데, 이때 사용할 수 있는 게 이전에 배웠던 context API였다.

그리고 Typescript는 자바스크립트보다 강력한 객체지향 프로그래밍의 기능들을 담고 있다. Typescript의 **interface**는 우리가 만들고자 하는 모듈의 명세서로 어떠한 상태와 행동들을 하는 모듈인지 정하고, 그것을 implements하는 class를 만들 수 있다.

### 📖 Interface

이번 과제에서 만들었던 class는 두가지 광고 데이터를 api로 불러오는 모듈이 필요했다. 그래서 interface는 다음과 같은 두 가지 함수를 가지고 있다.

```typescript
type GetAdListResponse = {
  ads: AdType[];
  counts: number;
};

type GetTrendResponse = {
  report: {
    daily: TrendType[];
  };
};

interface AdService {
  getAdList: () => Promise<GetAdListResponse>;
  getTrend: () => Promise<GetTrendResponse>;
}
```

API 통신을 위한 class이므로 각 메소드는 통신결과인 promise를 반환하고 promise 내부의 데이터 타입을 정해주면 interface를 작성할 수 있다.

### 📁 Class

우리가 정의한 interface를 구현하는 class인 AdserviceImpl를 다음과 같이 만들 수 있다.

```typescript
import { AxiosError, AxiosInstance } from 'axios';
import {
  AdService,
  GetAdListResponse,
  GetTrendResponse,
} from 'models/interface';
import HTTPError from '../network/httpError';

const AD_LIST_URL = '/ad-list-data-set.json';
const AD_TREND_URL = '/trend-data-set.json';

class AdServiceImpl implements AdService {
  constructor(private axiosInstance: AxiosInstance) {}

  getAdList = async () => {
    try {
      const { data } = await this.axiosInstance.get<GetAdListResponse>(
        AD_LIST_URL
      );
      return data;
    } catch (error) {
      const { response } = error as unknown as AxiosError;
      if (response) {
        throw new HTTPError(response?.status, response?.statusText);
      }
      throw new Error('Unknown Error');
    }
  };

  getTrend = async () => {
    try {
      const { data } = await this.axiosInstance.get<GetTrendResponse>(
        AD_TREND_URL
      );
      return data;
    } catch (error) {
      const { response } = error as unknown as AxiosError;
      if (response) {
        throw new HTTPError(response?.status, response?.statusText);
      }
      throw new Error('Unknown Error');
    }
  };
}

export default AdServiceImpl;
```

이전부터 항상 사용해오던 HTTPError class는 원하는 에러메시지를 커스텀할 수 있게 하는 정도로 사용했지만 에러가 다양해지고 복잡해지면 class에 내용만 추가하면 되니까 훨씬 유지 보수에 유용할 것이란 예상을 할 수 있다.

```typescript
export default class HTTPError extends Error {
  constructor(private statusCode: number, public message: string) {
    super(message);
  }

  get errorMessage() {
    switch (this.statusCode) {
      case 404:
        this.message = '잘못된 요청입니다. url을 확인해주세요';
        break;
      default:
        throw new Error('Unknown Error');
    }
    return this.message;
  }
}
```

### 💊 Dependency Injection

위의 만든 AdService를 사용하기 위해서는 instance를 만들고 instance를 context API로 전달해주면 필요한 곳에서 편하게 사용할 수 있다. typescript로 contextAPI로 instance를 전달할 때 어떻게 type을 전달해야 할 지 해결하는 와중에, 우리가 **정의한 interface**를 그대로 전달해주면 type을 간단하게 정해줄 수 있다는 점을 알게 되었다. interface를 기준으로 class를 만드는 **의존성 역전 원칙**을 이해하는 경험이었다.

[adService의 type을 interface로 전달해주기 전의 에러]

<img width="800" src="./context에 class instance를 넣으려하니 생긴에러.PNG"/>

여기서 조금 주의할 부분은 전달하면서 instance가 this를 잃어버린다는 점이다. 그래서 bind로 adService를 binding을 해주거나, method 자체를 **arrow function**으로 바꾸게 되면, this binding 문제를 해결할 수 있다.

```tsx
//index.tsx
const BASE_URL = process.env.REACT_APP_BASE_URL || '';
const axiosInstance = createAxiosClient(BASE_URL);
const adService = new AdService(axiosInstance);

const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);

root.render(
  <React.StrictMode>
    <AdServiceProvider adService={adService}>..</AdServiceProvider>
  </React.StrictMode>
);

//AdServiceContext.tsx
import { AdService } from 'models/interface';
import { createContext, useMemo, useContext } from 'react';

const AdServiceContext = createContext<AdService | null>(null);
export const useAds = () => useContext(AdServiceContext);

export const AdServiceProvider = ({
  children,
  adService,
}: {
  children: React.ReactNode;
  adService: AdService; //interface로 바로 type정해줄 수 있어
}) => {
  const { getAdList, getTrend } = adService;
  const value = useMemo(() => {
    return { getAdList, getTrend };
  }, [getAdList, getTrend]);
  return (
    <AdServiceContext.Provider value={value}>
      {children}
    </AdServiceContext.Provider>
  );
};
```

## 🛠React Query를 이용한 Refactoring

기능을 우선으로 코드를 작성하다 보니 너무 비슷한 코드가 두 번이나 사용되었는데, 코드 양이 너무 많아 가독성이 떨어지는 문제를 가지고 있었다.

```tsx
const getAdList = useCallback(async () => {
    listDispatch({ type: DataActionEnum.SET_IS_LOADING, isLoading: true });
     try {
      const response = _await_ adService?.getAdList();
      listDispatch({
        type: DataActionEnum.SET_DATA,
        data: response?.ads || [],
      });
    } catch (e) {
      console.error(e);
    } finally {
      listDispatch({ type: DataActionEnum.SET_IS_LOADING, isLoading: false });
    }
  }, [adService, listDispatch]);

const getAdTrend = useCallback(async () => {
    trendDispatch({ type: DataActionEnum.SET_IS_LOADING, isLoading: true });
    try {
      const response = _await_ adService?.getTrend();
      trendDispatch({
        type: DataActionEnum.SET_DATA,
        data: response?.report.daily || [],
      });
    } catch (e) {
      console.error(e);
    } finally {
      trendDispatch({ type: DataActionEnum.SET_IS_LOADING, isLoading: false });
    }
  }, [adService, trendDispatch]);
```

dispatch함수와 action내용만 바뀌는 두 가지 함수 내용을 보면서 하나의 함수로 만들어서 전달할 까도 생각했지만 Type과 관련해서 너무 복잡해지는 문제를 가지고 있었다. 코드의 간결화와 두 가지 분리된 함수를 가지는 게 더 좋을 것 같아 React Query를 사용했다.

## React Query

<img width="1000" src="https://velog.velcdn.com/images/woohobi/post/0233c2ea-03ab-439f-b735-7bb125f091f0/image.svg"/>

React Query는 서버에서 데이터를 받아오고, 받아온 데이터를 caching, retry 등 비동기에 필요한 다양한 기능을 제공해 주는 라이브러리다. React Query는 이후에 상태 관리 라이브러리와 연동한다면 상태는 상태를 보관하고 변경하는 역할만, React Query로는 서버에서 받아온 데이터만을 관리하는 역할만 분리해서 사용할 수 있을 것 같아 공부하고 적용해보았다.

위 상황은 두 가지 API 함수로 호출을 하는 상황이고 각각의 데이터를 상태로 전달해줘야 하는 상황이었다. (React query가 나서기 딱 좋은 상황이다)

이를 받아온 결과들만 뽑아서 상태를 전달하니, 훨씬 코드가 간단해지고 구분이 잘된 것을 볼 수 있다.

```tsx
const { isLoading, data: trendData } = useQuery(
  ['trend'],
  () => adService?.getTrend(),
  {
    staleTime: 1000 * 60 * 60,
    cacheTime: 1000 * 60 * 60,
  }
);
const { data: listData } = useQuery(['adList'], () => adService?.getAdList(), {
  staleTime: 1000 * 60 * 60,
  cacheTime: 1000 * 60 * 60,
});

useEffect(() => {
  trendDispatch({
    type: DataActionEnum.SET_DATA,
    data: trendData?.report.daily || [],
  });
  trendDispatch({ type: DataActionEnum.SET_IS_LOADING, isLoading });
}, [trendData, isLoading]);

useEffect(() => {
  listDispatch({
    type: DataActionEnum.SET_DATA,
    data: listData?.ads || [],
  });
}, [listData]);
```

## 😅Typescript Error

역시 Typescript는 아직 어렵다... 마주했던 에러들을 기록해 이후에 까먹을 내가 참고하려한다.

### Object Key Type

저번 과제에서 Detail 페이지를 구성할때 api로 받은 데이터 중에서 param값과 같은 id를 갖는 데이터를 찾아서 가져왔다. 이렇게 할 때 만약에 데이터가 커진다면 찾는데 훨씬 오랜 시간이 걸리는 문제가 발생한다 (배열에서 찾기는 O(n)). 그래서 Object로 자료구조를 바꾸려 했었는데 Type을 어떻게 설정할지 몰라 포기했던 경험이 있다.

이번에는 데이터를 계산하면서 Object.keys() 메소드를 이용했는데, 나열하려면 index의 type을 결정해줘야 했다.

```typescript
export type ResultType = {
  [index: string]: number;
};

const calculateData = (data: TrendType[]) => {
  const result: ResultType = {
    imp: 0,
    click: 0,
    cost: 0,
    conv: 0,
    convValue: 0,
    ctr: 0,
    cvr: 0,
    cpc: 0,
    cpa: 0,
    roas: 0,
  };

  data.forEach((item) => {
    Object.keys(item).forEach((key) => {
      if (key in result) {
        result[key] += Number(item[key]);
      }
    });
  });
  return result;
};

export default calculateData;
```

만약 저번 과제에서 해결 못했던 Object의 key로 데이터의 id를 사용하고 데이터를 value로 한 자료구조의 type 경우는 다음과 같이 적용할 수 있다.

```typescript
type Type = {
  [index: string]: { data: string };
};

const obj: Type = {
  '123123': { data: 'hi' },
};
```

# 😥아쉬웠던 점

이번 과제는 너무 디테일한 UI들과 selector를 이용해 받은 데이터들을 필터 해야 하는 부분들이 많아, 완전히 구현하지 못했다. 필터링 로직을 많이 구현하다 시간이 없어 그래프의 형식들도 신경 쓰지 못했고, adList 데이터로 수정하기 부분도 구현하지 못했다. 아직 너무 많이 부족하다는 것을 많이 느낀다. 코딩을 하면서 저번에 했던 코드를 복사해 수정해서 사용하다 보니, "정말 내가 이 코드들을 제대로 이해하고 있는 걸까"라는 생각도 들고, 부족함에 좌절하기도 많이 했다.

그래도 매번 과제를 하면서 내가 가진 장점은, 프로젝트 내의 **문제점을 잘 찾는다**는 점과 **배운 것을 적용할 수 있는 능력**이라고 생각한다. 내 프로젝트에서 부족한 점들을 기록해서 최대한 채웠고, 멘토님이 말씀해 주셨던 부분들을 담아서 코딩하다 보니 이전과는 다른 시야와 다른 수준으로 코딩하고 있다는 생각을 한다.

좌절하고 힘들었지만 거기서 멈추지 말고 이번 부족한 부분들을 밑거름으로 다음으로 나아가야겠다.
