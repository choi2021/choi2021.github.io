---
title: '원티드 프리온보딩 2-1 과제회고'
date: 2022-11-04
slug: wanted-pre-onboarding-2-1
tags: [회고, 원티드프리온보딩]
series: 원티드프리온보딩
---

# 📜 과제 설명

이번 과제는 기업과제로 주어진 피그마의 디자인과 api를 이용해 2가지 페이지를 구현해야했다. 페이지는 차량리스트를 보여주는 Home페이지, 해당 차량의 정보를 보여주는 detail페이지로 구성되어 있으며, 추가 구현사항으로 페이스북과 카카오톡에 공유시 해당 이미지와 차량정보들을 보여줄 수 있어야하는 **SEO**가 있었다. 과제 자체는 저번 과제와 크게 다른 점이 없어서 수월하게 할 수 있을 것 같아, 이번 기회에 모두 다같이 **typescript**를 도입해보기로 했다.

## UseReducer와 Context API

처음 과제부터 계속해서 사용해와서 조금은 익숙해진 context API와 useReducer를 이번에 함께 사용해보았다. 다른 팀의 저번과제의 코드들과 [Velopert님의 글](https://react.vlpt.us/)을 참고해서 코드를 구성했다.

### UseReducer

useReducer는 중첩된 상태나 여러가지 상태를 하나의 오브젝트로 묶어서 관리할 때 등, 복잡한 상태관리 로직을 간단하게 처리할 수 있는 react hook이다. useReducer의 로직은 useState와 유사하게, 우리가 관리해야 할 **상태**가 있고, 상태를 어떻게 처리할지를 담고 있는 **action**과 전달받은 action에 따라 처리해주는 **dispatch**가 있다.

```jsx
const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case ActionType.SET_IS_LOADING:
      return {
        ...state,
        isLoading: action.isLoading,
      };
    ...
};

const [state, dispatch] = useReducer(reducer, initialState);
// const [state,setState]=useState()와 유사해
```

이번 프로젝트에서 reducer를 사용해본 부분은 API호출에 따른 error, isLoading, data를 하나로 관리하기 위해, 저번 usefetch로 분리했던 customHook을 useReducer로 대체했다.

```typescript
type State = {
  isLoading: boolean;
  data: CarType[];
  error: string;
};

type Action =
  | { type: ActionType.SET_DATA; data: CarType[] }
  | { type: ActionType.SET_IS_LOADING; isLoading: boolean }
  | { type: ActionType.SET_ERROR; error: string };

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case ActionType.SET_IS_LOADING:
      return {
        ...state,
        isLoading: action.isLoading,
      };
    case ActionType.SET_DATA:
      return {
        ...state,
        data: action.data,
      };
    case ActionType.SET_ERROR:
      return {
        ...state,
        error: action.error,
      };
    default:
      throw new Error('Unknown Action');
  }
};

export const CarsProvider = ({ children }: { children: React.ReactNode }) => {
  const [state, dispatch] = useReducer(reducer, initialState);
		...
};
```

### contextAPI

contextAPI를 기존에서 사용할 때는 value에 상태와 함수를 같이 보내주었지만 이번에 reducer를 사용하면서 상태와 dispatch 둘 중 하나만 필요할 때가 있어, stateContext와 dispatchContext 두 가지로 나누어서 구성했다.

```tsx
type State = {
  isLoading: boolean;
  data: CarType[];
  error: string;
};

type Action =
  | { type: ActionType.SET_DATA; data: CarType[] }
  | { type: ActionType.SET_IS_LOADING; isLoading: boolean }
  | { type: ActionType.SET_ERROR; error: string };

type CarsDistpatch = Dispatch<Action>;

export const CarsStateContext = createContext<State | null>(initialState);
export const CarsDispatchContext = createContext<CarsDistpatch | null>(null);

export const CarsProvider = ({ children }: { children: React.ReactNode }) => {
		...
    return (
    <CarsStateContext.Provider value={state}>
      <CarsDispatchContext.Provider value={dispatch}>
        {children}
      </CarsDispatchContext.Provider>
    </CarsStateContext.Provider>
  );
};
```

useReducer와 contextAPI를 이용해서 보다 깔끔하게 상태관리를 할 수 있었고, reducer에서만 상태관리 로직을 추가하면 되어서 확장성도 좋은 장점을 갖게 되었다.

```tsx
enum ActionEnum {
  SET_IS_LOADING = 'SET_IS_LOADING',
  SET_DATA = 'SET_DATA',
  SET_ERROR = 'SET_ERROR',
}

const App = () => {
  const dispatch = useCarsDispatch();
  const getList = useCallback(async () => {
    dispatch({ type: ActionType.SET_IS_LOADING, isLoading: true });
    try {
      const response = await carsAPI.getCars();
      if (response) {
        dispatch({ type: ActionType.SET_DATA, data: response?.payload });
      }
    } catch (e) {
      if (e instanceof HTTPError) {
        dispatch({ type: ActionType.SET_ERROR, error: e.errorMessage });
      }
      console.error(e);
    } finally {
      dispatch({ type: ActionType.SET_IS_LOADING, isLoading: false });
    }
  }, [dispatch]);
  useEffect(() => {
    getList();
  }, [getList]);

  return (
    <>
      <Header />
      <Outlet />
    </>
  );
};

export default App;
```

### contextAPI를 이용한 Filtering

이번 과제에서 전체 차량중에서 category를 누르면 해당 차량의 종류만 보여줘야했기 때문에 filtering 로직도 필요했다. filtering을 하기 위해서는 기존의 상태를 가지고 있으면서 filter하고 싶은 차량들만 보여줘야 했기 때문에 기존 Reducer로직에 추가하지 않고 따로 caterogryContext를 만들어 관리했다.

```tsx
//categoryContext.tsx
import { createContext, useState, useMemo } from 'react';
import { CategoryType } from 'types/CarsInterface';

const initialState = {
  category: '전체',
  setCategory: (category: CategoryType) => {},
};

export const CategoryContext = createContext(initialState);

export const CategoryProvider = ({
  children,
}: {
  children: React.ReactNode;
}) => {
  const [category, setCategory] = useState<CategoryType>('전체');
  const value = useMemo(() => ({ category, setCategory }), [category]);
  return (
    <CategoryContext.Provider value={value}>
      {children}
    </CategoryContext.Provider>
  );
};
```

각각의 context API의 provider는 필요한 곳에서 감싸 주려했다. 차량 목록이 있다면 useParam으로 해당 차량 정보도 얻을 수 있기 때문에 따로 api를 호출하지 않고 한번만 호출하게 하기 위해 Router.jsx에서 carsProvider를 감싸주었다. categoryProvider는 category를 update하고 category를 이용해 filtering된 결과를 받아오기 위해 categories와 carsList가 있는 home.tsx에서 감싸주었다.

```tsx
//router.tsx
const Router = () => {
  return (
    <CarsProvider>
      <RouterProvider router={router} />
    </CarsProvider>
  );
};

//

const Home = () => {
  return (
    <CategoryProvider>
      <S.Section>
        <Categories />
        <CarList />
      </S.Section>
    </CategoryProvider>
  );
};

export default Home;
```

## Custom Hook

이번 과제를 하면서 가장 신경썼던 포인트중 하나는 **컴포넌트의 단순화**였다. 멘토님께서 강의 해주신 컴포넌트의 추상화에 대해 많이 생각하면서 되도록이면 Component가 로직과 관련된 코드를 많이 가지고 있지 않고, UI 렌더링 로직만을 가지고 있게 노력했다. 그렇게 하기 위해서는 중복되거나 사용되는 로직을 다른 파일로 보관해야 했고, **custom hook**을 적극적으로 사용했다.

특히 home page의 carsList 컴포넌트는 api로 받아온 차량리스트를 카테고리에 맞게 보여줘야했다. 내부에 carsContext로부터 받아온 데이터를 filtering을 할 수도 있지만 **로직을 컴포넌트 내부에 남기고 싶지 않아** customHook으로 만들어 list만 받아올 수 있게 했다. useCarsValue 내부에서 필터링하기 때문에 컴포넌트는 엄청 간소화된 구조를 가질 수 있었다.

```tsx
//useCars.tsx

export const useCarsState = () => {
  const state = useContext(CarsStateContext);
  if (!state) throw new Error("Can't find State Provider");
  return state;
};

export const useCarsDispatch = () => {
  const dispatch = useContext(CarsDispatchContext);
  if (!dispatch) throw new Error("Can't find Dispatch Provider");
  return dispatch;
};

export const useCarsValue = () => {
  const state = useCarsState();
  const { category } = useContext(CategoryContext);

  if (!state) throw new Error("Can't find StateProvider");
  if (!category) throw new Error("Can't find CategoryProvider");
  if (category === '전체') return state.data;

  const filterd = state?.data.filter(
    (car) => SegmentEnum[car.attribute.segment] === category
  );
  return filterd;
};

//carsList.tsx
import S from './styles';
import CarItem from '../carItem/CarItem';
import { useCarsState, useCarsValue } from '../../hooks/useCars';

const CarList = () => {
  const { isLoading, error } = useCarsState();
  const data = useCarsValue();
  if (isLoading) {
    return (
      <S.Layout>
        <h3>불러오는 중</h3>
      </S.Layout>
    );
  }

  if (error) {
    return (
      <S.Layout>
        <h3>{error}</h3>
      </S.Layout>
    );
  }

  if (data.length === 0) {
    return (
      <S.Layout>
        <h3>차량이 없습니다.</h3>
      </S.Layout>
    );
  }
  return (
    <ul>
      {data.map((car) => (
        <CarItem key={car.id} {...car} />
      ))}
    </ul>
  );
};

export default CarList;
```

## Typescript

typescript는 공부를 해도 잘 쓰는 법이 무엇인지 고민이 많이 되었기 때문에 빠른 개발을 위해서 react로 한 후에 천천히 typescript로 바꿔야지라고 생각했지만, 계속 미뤄왔왔다. 이제부터는 계속해서 사용하면서 부딪히면서 배워나가기로 마음먹었다. 이번 과제는 너무 친절하게 과제의 api문서에 데이터마다 type까지 자세히 알려주기 때문에 꼭 적용해보고 싶어 적극적으로 팀에 제안했다.

### enum

enum은 비슷한 역할을 하는 변수들을 묶음으로 최대한 string이나, number인 상태로 의미를 알 수 없는 코드를 남기지 않으려 사용했다. enum을 사용할 때 새롭게 알게된 점은 object와 같이 사용이 가능하다는 점이었다.

```typescript
enum SegmentEnum {
  C = '소형',
  D = '중형',
  E = '대형',
  SUV = 'SUV',
}

type AttributeType = {
  segment: keyof typeof SegmentEnum;
};
```

segment의 type을 전달할 때 segmentEnum중의 하나라고 알려줄 때 **keyof typeof**를 이용할 수 있었고 이렇게 전달해준 enum의 value값을 찾을 때는 custom Hook에서 key값을 전달해서 찾을 수 있었다.

```tsx

export const useCarsValue = () => {
    //	...
  const filterd = state?.data.filter(
    (car) => SegmentEnum[car.attribute.segment] === category
  );
  return filterd;
};

```

### null/undefined error

null/undefined Error는 아마 가장 자주 마주하는 에러가 아닐까 싶다. 조건부로 받아올 경우나 null로 받아올 경우 해당 오브젝트의 property가 없을 수도 있기 때문에 에러를 던져준다.

<img width="800" src="./object null error.PNG"/>

에러를 막기위해서는 항상 undefined이나 null일 경우에 처리할 수 있는 로직을 처리하면 간단하게 해결이 가능하다.

```tsx

const Detail = () => {
  const { id } = useParams();
  const car = data.find((item) => item.id === +id);

    //	...

  if (!car) {
    return (
      <S.Layout>
        <h3>url을 확인해주세요</h3>
      </S.Layout>
    );
  }

  const { amount, attribute, startDate, insurance, additionalProducts } = car;

 // ..
};

export default Detail;

```

## CRA에서의 SEO 문제 해결

이번과제를 할 때 CRA에서 간단하게 react-helmet을 이용하면 SEO를 해결할 수 있을 것이란 생각에 CRA를 이용해서 진행했다. 하지만 마주한 문제들이 많았는데 문제해결과정을 정리해보고자 한다.

### SEO 관점에서의 CSR과 SSR

<img width="800" src="https://velog.velcdn.com/images%2Fhanei100%2Fpost%2F138d8a8d-8df9-4bca-bf0f-561d01b4cc84%2Fimage.png"/>

CSR은 client에서 화면을 렌더링하는 방식으로, 서버에서 받은 **하나의 빈페이지 index.html**에 동적으로 html 요소를 만드는 javascript을 받아 한번에 보여준다. 그로인해 화면이 보임과 동시에 interactive한 페이지를 만들 수 있는 장점이 있다. 내가 자주 사용하는 CRA (create-react-app)는 간편하게 CSR (client-side-rendering)이 가능한 패키지이지만 CSR의 특성으로 SEO에는 취약한 단점을 가진다.

<img width="800" src="https://velog.velcdn.com/images%2Fhanei100%2Fpost%2F040f9b95-70a0-49fa-8b73-76272597890c%2Fimage.png"/>

그에 반해 SSR (server-side-rendering) 은 서버에서 정적인 페이지를 먼저 만들어 렌더링해주기 때문에 초기 렌더링 속도가 빠르고 검색엔진과 같은 봇이 보았을 때 해당 내용들을 볼 수 있기 때문에 SEO에 큰 장점을 갖고 있다. CSR에 비해 먼저 화면이 보이고 이후에 javascript가 실행되기 때문에 ux측면에서는 단점을 가질 수 있다. 이번 과제를 위해서는 SSR이 더 적합한 방식이었을 것이란 생각이 된다.

### 그러면 왜 CRA로 진행했을까?

React에서 SSR을 하기 위해서는 Next.js를 사용하면 된다. 하지만 아직 사용해본 적이 없고, typescript에 좀더 초점을 맞춰서 공부하다 보니 시간이 부족해 우선 어떻게든 CRA에서 해결할 수 있는 방법을 찾아서 적용해보았다.

## React-Helmet

react-helmet은 react 라이브러리로 index.html의 head 내용을 동적으로 바꿀 수 있는 라이브러리이다. 과제에 필요한 내용들을 각 detail 페이지의 정보에 맞게 head 내용을 바꾸기 위해 meta 컴포넌트를 만든 후에 정보를 담아주었다. 그결과 페이지에서 잘바뀌어있는 것을 확인할 수 있었다.

<img width="800" src="https://user-images.githubusercontent.com/104304569/199743653-4db5c757-19ed-44d5-b238-1a94bc88a255.png"/>
<br/>

```jsx
import { Helmet } from 'react-helmet-async';

const Meta = ({ attribute, amount, id }: MetaProps) => {
  const { brand, name, imageUrl } = attribute;
  return (
    <Helmet>
      <title>{`${brand} ${name}`}</title>
      <meta name="description" content={`월 ${amount}원`} />
      <meta property="og:type" content="website" />
      <link href={imageUrl} />
      <meta property="og:url" content={`${process.env.PUBLIC_URL}/${id}`} />
      <meta name="og:title" content={`${brand} ${name}`} />
      <meta name="og:description" content={`월 ${amount.toLocaleString()}원`} />
      <meta property="og:image" content={imageUrl} />
      <meta property="og:image:width" content={IMAGE_SIZE.width.toString()} />
      <meta property="og:image:height" content={IMAGE_SIZE.height.toString()} />
    </Helmet>
  );
};
```

하지만 공유를 할때는 여전히 초기 index.html의 head내용만 보이는 문제점이 존재했다. 이러한 문제점은 head내용이 javascript를 이용해 동적으로 바뀌지만 공유를 했을 때는 하나의 index.html의 내용이 그대로 반영되어 생긴 문제로 생각됐다.

### React-snap

react-snap은 react library로 react-router로 만든 동적라우팅 페이지마다 적합한 html파일을 만들어주는 라이브러리이다. index.tsx를 hydrate를 이용해 client-side페이지를 static HTML로 바꿔준다. 바꿔준 결과 build폴더에 만들어질 페이지들의 폴더와 index.html이 생긴 걸 볼 수 있다.

<img width="500" src="https://user-images.githubusercontent.com/104304569/199749072-fc686c18-6bec-4dfc-81ce-2666386d4d2c.png"/>

```tsx
import { hydrate, render } from 'react-dom';

const container = document.getElementById('root') as HTMLElement;
const root = ReactDOM.createRoot(container);

if (container.hasChildNodes()) {
  ReactDOM.hydrateRoot(
    container,
    <React.StrictMode>
      <ThemeProvider theme={Theme}>
        <GlobalStyle />
        <Router />
      </ThemeProvider>
    </React.StrictMode>
  );
} else {
  root.render(
    <React.StrictMode>
      <ThemeProvider theme={Theme}>
        <GlobalStyle />
        <Router />
      </ThemeProvider>
    </React.StrictMode>
  );
}
```

두가지 라이브러리를 이용한 덕분에 다행히 공유시 내용들이 담아 문제를 해결할 수 있었고, react-snap을 쓰면서 알게된 **hydration**이란 방식이 실제로 SSR을 위한 프레임워크들 Next.js와 Gatsby가 이용하는 방식임을 알게 되었다.

<img width="300" src="https://user-images.githubusercontent.com/104304569/199750059-845a9e40-679c-43cf-bff8-eac249801b9a.png"/>

<img width="300" src="https://user-images.githubusercontent.com/104304569/199750370-7cc423bd-0972-4fba-997a-69d32788829e.png"/>

## Axios

이번 프로젝트를 하면서 새롭게 시도한 것은 fetch대신 axios를 사용했다는 점이었다. fetch로 잡지 못했던 에러들을 request와 response로 나눠서 받아 에러핸들링이 더 간편했으며, 팀원분이 알려주신 axios의 interceptor를 이용하면 보내기 전에 설정들을 추가할 수도 있어 더 유용한 부분이 많은 라이브러리라 생각되었다.

원래는 하나의 api만 사용할 때는 class로 사용하면 오히려 더 복잡하게 만든다고 생각해서 사용하지 않았지만 class로 분리하면 좀 더 정리가 될 수 있고 확장성이 높다는 장점이 있고, 전달시 instance를 만들어서 사용하는 점을 배울 수도 있었다.

```ts
import axios, { AxiosError, AxiosInstance } from 'axios';
import { CarType, FuelEnum, SegmentEnum } from 'types/CarsInterface';
import createAxiosInstance from './axiosUtils';
import HTTPError from '../network/httpError';

const BASE_URL = 'https://preonboarding.platdev.net/api/cars';

type GetCarsResponse = {
  payload: CarType[];
};

class CarsAPI {
  constructor(private axiosInstance: AxiosInstance) {}

  async getCars(fuelType?: FuelEnum, segment?: SegmentEnum) {
    try {
      const { data } = await this.axiosInstance.get<GetCarsResponse>(BASE_URL, {
        params: {
          fuelType,
          segment,
        },
      });
      return data;
    } catch (error) {
      const { response } = error as unknown as AxiosError;
      if (response) {
        throw new HTTPError(response?.status, response?.statusText);
      }
      throw new Error('Unknown Error');
    }
  }
}

const carsAPIinstance = createAxiosInstance(BASE_URL);

const carsAPI = new CarsAPI(carsAPIinstance);

export default carsAPI;
```

### 에러핸들링

기존에 사용했던 에러핸들링을 위한 class를 이용해서 필요한 에러 메시지를 좀 더 이해가 잘되게 수정했고, typescript에서 제공하는 private, public을 이용해 보다 간단하게 constructor함수를 사용할 수 있었다. 에러를 상속받기 때문에 message는 public으로 사용해줘야 된다는 점을 새롭게 알 수 있었다.

```ts
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

### 📢 마치며

이번 기회를 통해서 왜 기업이 SEO를 고려하는지, SEO를 해결하는 방식으로 SSR을 이용해야하는 이유를 체감할 수 있었다. 무조건 SSR이 좋고 유행하니까 해야된다는 생각보다 어떤 문제를 해결하기 위해 나왔고 이것을 어디에 적용해야하는 지 알게되었고, Next.js를 공부해보고 싶다는 의욕이 더 생기는 계기가 되었다. 하나를 완벽하게 하고 다음을 해야하지 않나 생각도 하지만, 이번에 next.js를 몰라서 적용을 못했다는 아쉬움이 생겨, **내가 사용할 수 있는 도구를 늘이는 과정도 필요하다**는 깨우침도 생겼다.
