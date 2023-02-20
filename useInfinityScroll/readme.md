# Infinity Scroll

사용자가 일정 스크롤을 했을때, 밑에 데이터가 추가로 붙음.

페이지의 특정 지점을 스크롤 했을 때 새 데이터를 가져오게 한다.

<aside>
💡 사용자가 데이터의 하단으로 오면 새로운 데이터를 가져와서, 사용자가 중단 없이 계속 스크롤 할 수 있게 해줌.

</aside>

하나의 쿼리에 계속 추가 된다.

기본 셋팅

```jsx
function App() {
  const queryClient = new QueryClient();
  return (
    <QueryClientProvider client={queryClient}>
      <div className='App'>
        <h1>Infinite SWAPI</h1>
        <InfinitePeople />
        {/* <InfiniteSpecies /> */}
      </div>
      <ReactQueryDevtools />
    </QueryClientProvider>
  );
}
```

<br>

## UseInfinityQuery

- vs useQuery ⇒ {data} 는 **단순히 쿼리 함수에서 반환되는 ‘데이터’** 였다.
- useQuery와 다르게 useInfinityQuery에서 **{ data }객체는 두 개의 프로퍼티**를 가지고 있다.

![스크린샷 2023-02-15 오후 12.53.12.png](Infinity%20Scroll%206318fa7d0f704ec287d1d7b2b10c32e5/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-02-15_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_12.53.12.png)

1. `pages` : 데이터의 안의 정보, pages하나당 useQuery에서 받는 데이터에 해당한다.
2. `pageParams` : 각 페이지의 매개변수가 기록되어있다. 검색된 쿼리의 키를 추적한다. (흔하게 사용되지 않는다…??)

<br>

## Syntax

- pageParam은 쿼리 함수에 전달되는 매개변수이다.

```jsx
// 기본 문법
useInfiniteQuery('sw-people', ({ **pageParam** = defaultUrl }) => fetchUrl(**pageParam**))
```

- defaultUrl : fetchUrl에 처음에 전달할 초기값 셋팅
- `fetchUrl()`이 pageParam을 사용해서 적절한 페이지를 찾아온다.
- 기본 page-nation과는 다르게 React-query가 pageParam의 현재 값을 유지한다.??
- useInfinitequery : options들
  - `getNextPageParam` : (lastPage, allPages) : 다음 페이지로 가는 방식을 정의하는 `함수`
    - lastPage : 마지막 페이지의 데이터
    - allPages : 모든 페이지에 대한 데이터
    - pageParam을 업데이트 해줄 수 있다.
    - 현재 예시에서는 lastPage를 사용할 것이고, 구체적으로 말하면 `next` property이다.
    - Star Wars API는 데이터의 다음 페이지를 가져오려면 어느 함수를 실행해야할지 알려준다.
- useInfinitequery가 반환하는 객체 종류들

  - fetchNextPage : 사용자가 더 많은 데이터를 요청할 때 호출하는 함수, 더 많은 데이터를 요청하는 버튼을 클릭.
  - hasNextPage : `getNextPageParam` 함수의 반환 값을 기반으로 한다. 마지막 쿼리의 데이터를 어떻게 사용할지 지시한다.
    - undefined인 경우 더 이상 데이터가 없는 것이고,
  - isFetchingNextPage : react-query는 일반적인 fetching인지 다음 페이지를 fetching하는지 구분을 할 수 있다.

  <br>

## 인피니트 플로우

1. 컴포넌트가 마운트 된다.
2. useInfinityQuery에서 반환되는 { data } 는 undefined이다. 아직 페칭해오지 않았기 때문!
3. useInfinityQuery의 쿼리 function을 이용해 첫 페이지를 가져온다.
   1. useInifiniteScroll({ pageParam = defaultUtl(기본값) } ⇒ … ) `pageParam`을 인수로 받는다.
   2. 첫번째 pageParam은 이 요소에서 **우리가 기본값으로 정의한 것이 된다.**
   3. 첫번째 pageParam을 사용해서 첫 번째 페이지를 가져온다.
   4. {data} 객체의 pages 프로퍼티를 취득할 수 있다.
   5. data.pages[0]의 데이터들을 가져온다. === 쿼리 함수가 반환하는 값
   6. 첫번째 데이터가 반환된 후 React-query가 `getNextPageParam`을 실행한다.
   7. getNextPageParam은 lastPage와 allPages를 사용해서 pageParam을 업데이트한다.
   8. hasNextPage로 다음페이지가 있는지 판별할 수 있고, hasNextPage가 결정되는 방식은 pageParam이 정의되어 있는지 아닌지에 따라이다.
      1. pageParam이 정의되어있으면, 다음 페이지가 있는 것
   9. 사용자가 스크롤을 활용하거나, 버튼을 클릭해서 `fetchNextPage`함수를 트리거 하는 행동을 하였다면?
   10. 이때 React-query가 이 쿼리 함수를 실행시킨다.({ pageParam = defaultUtl(업데이트 된 값) } ⇒ … )
   11. pages 배열에 다음 데이터를 추가 하게 된다. 새로운 데이터가 배열에 추가된다.
   12. 새로운 데이터를 가지고 `getNextPageParam`을 실행해서 다음 pageParam을 업데이트 한다. 하지만 데이터가 총 2페이지까지만 있다고 가정할때, 다음 pageParam은 undefined이 된다( 더 이상 다음 페이지가 없으므로…)
   13. 다음 pageParam이 undefined 였기 때문에, hasNextPage는 `false` 된다. 더 이상 수집할 데이터가 없다….. 끝!!!!!

- 예시의 sw-api는 마지막 페이지에 그다음 page url 정보가 들어있다(next 프로퍼티) 그래서 getNextPageParam을 통해서 pageParam정보를 업데이트 해줄수 있다.

```jsx
{
  getNextPageParam: (lastPage) => lastPage.next;
}
```

- hasNextPages는 getNextPageParam이 undefined를 반환하는지 아닌지에따라 boolean값을 return 하게 된다.
- [lastPage.next](http://lastPage.next) 가 거짓(여기 예제에서는 null) 이면 undefined를 반환하게 한다.
- sw-api에서는 다음 페이지가 없을 경우 next 프로퍼티는 `null`을 반환하게 된다.

<br>

## react-inifinite-scroll 라이브러리

- react-inifinite-scroll 컴포넌트에는 두 개의 프로퍼티가 존재한다.
  1. loadMore props
     - 데이터가 더 필요할 때 불러와 useInifiniteQuery에서 나온 fetchNextPage 함숫값을 이용한다.
  2. hasMore props
     - hasNextPage를 넘긴다.
  3. 끝!!!

<aside>
💡 react-inifinite-scroll 의 InifiniteScroll 컴포넌트는 스스로 페이지의 끝에 도달했음을 인식하고, fetchNextPage를 불러오는 기능을 제공한다.

</aside>

<br>

## 인피니티 쿼리 작성후 data가 undefined가 되는 경우에 대해서

- 최초에 useInfiniteQuery를 실행할 때, **이 쿼리 함수가 해결될 때까지 정의되지 않았다고 반환하기 때문.**
- useQuery에서 했던 것 처럼 오류를 잡는 방법은???

<br>

## InifiniteQuery의 fetching 상태와 Error 상태 처리하는 법

```jsx
if (isLoading) return <div className='loading'>Loading......</div>;
if (isError) return <div>Error! {error.toString()}</div>;
```

- fetching 중일때 로딩 컴포넌트를 early return 하고
- Error가 있을때는 에러를 표시하는 컴포넌트를 return 해주었다.
- 해결을 되었지만, 다음 페이지 데이터를 불러올때, 데이터를 가져오고 있다는 피드백을 표시하려고 하면 어떻게 해야될까?!!??!

- isLoading → isFetching으로 바꿔본다

  - 다음페이지를 불러올때마다, 스크롤이 맨 위로 가버린다… → 새로운 페이지가 열릴 때마다 조기 반환이 되기 때문….

<br>

## 정리

- pageParam은 가져와야할 다음 페이지 정보를 가지고 있다…
  - 이 인자는 getNextPageParam옵션 함수에서 표시 할 수 있다.
  ```jsx
  {
    getNextPageParam: (currentPage) => {
       return currentPage.posts.length > 0 ? currentPage.page + 1 : null
  },
  ```
  - lastPage,
  - pageParam은 hasNetPage 데이터를 더 불러올 수 있는지 없는지 판단.
  - fetchNextPage는 실제 데이터를 불러오는 것
