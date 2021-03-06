#  Server Integration
## References 
[whatwg-fetch - Library to simplify web requests](https://github.com/whatwg/fetch)

## Hook up to the remote to referesh data
``` 
yarn add whatwg-fetch
```
Update **src/actions.js**
```diff 
export const CHANGE_SEARCH_STRING = 'react101/CHANGE_SEARCH_STRING';
export const START_SEARCH = 'react101/START_SEARCH';
+ export const CHANGE_MOVIEDB = 'react101/CHANGE_MOVIEDB';

export function changeSearchString(searchString) {
  return {
    type: CHANGE_SEARCH_STRING,
    searchString,
  };
}

export function startSearch() {
  return {
    type: START_SEARCH,
  };
}


+ export function changeMovieDB(movies) {
+  return {
+    type: CHANGE_MOVIEDB,
+    movies
+  };
+ }
```

Update **src/sagas.js**
```diff
+ import { changeMovieDB, START_SEARCH } from "./actions";
- import { START_SEARCH } from "./actions";
+ import { takeEvery, call, put, select } from "redux-saga/effects";
- import { takeEvery } from "redux-saga/effects";

+ import 'whatwg-fetch';
+ import { makeSelectSearchString } from "./selectors";

+ function parseJSON(response) {
+   return response.json();
+ }

+ function checkStatus(response) {
+   if (response.status >= 200 && response.status < 300) {
+     return response;
+   }
+   const error = new Error(response.statusText);
+   error.response = response;
+   throw error;
+ }

+ export function get(url) {
+   const promise = fetch(url,
+     {method: 'GET'});
+   promise.then(checkStatus);
+   promise.catch(function(e) {
+     throw e});
+   return promise.then(parseJSON);
+ }


function* search()
{
+  const apiKey = "057dfa32a18eed0f2dc23dc2e80ed8a0";
+  const searchString = yield select(makeSelectSearchString());
+  const url = "https://api.themoviedb.org/3/search/movie?page=1&include_adult=false&language=en-US&api_key=" + apiKey + "&query=" + searchString;
+
+  try {
+    const data = yield call(get, url);
+    yield put(changeMovieDB(data));
+   } catch (ex) {}
-  console.log("search called");
}

export function* watchForSearchActions() {
    yield takeEvery(START_SEARCH, search);
}
```

Update **src/reducers.js**
```diff
import { combineReducers } from 'redux-immutable';
- import { fromJS } from 'immutable';
+ import { fromJS, Map } from 'immutable';
- import { CHANGE_SEARCH_STRING } from "./actions";
+ import { CHANGE_MOVIEDB, CHANGE_SEARCH_STRING } from "./actions";
import { reduce } from 'lodash/collection'

const initialState = fromJS({
  searchString: '',
  moviesDB: {},
});

+ const buildMovieMap = (movies) => reduce(movies, (result, movie) => result.set(movie.id, fromJS(movie)), new Map());

function movies(state = initialState, action) {
  switch (action.type) {
    case CHANGE_SEARCH_STRING:
      return state
        .set("searchString", action.searchString);
+    case CHANGE_MOVIEDB:
+      return state
+        .set("moviesDB", buildMovieMap(action.movies.results));
    default:
      return state
  }
}

export const moviesApp = combineReducers({
  movies
});
```

[Proceed to Step 10 - Testing](setup-testing.md)