<!--
name:		
title:		mergeMap vs exhaustMap vs switchMap vs concatMap
pageTitle:	RxJS mergeMap vs exhaustMap vs switchMap vs concatMap comparison with a marble diagram
desc:		See this head-to-head comparison of mergeMap vs exhaustMap vs switchMap vs concatMap 
docs:		
-->

Source that emits at **5ms, 10ms, 20ms** will be *`*Mapped`* to `timer(0, 3)`, limited to 3 emissions

```js
const { rxObserver, palette } = require('api/v0.3');
const { from, timer, pipe } = require('rxjs');
const { zip, take, map, mergeMap, switchMap, concatMap, exhaustMap, tap, delayWhen } = require('rxjs/operators');


// our source$ will emit values at 5ms, 10ms, 20ms
const source$ = fromDelayed([ 5, 10, 20 ]).pipe(
    zip(from(palette), Marble) // colorize each item
  );

// target$ that we'll be mapping to
const target$ = timer(0, 3).pipe(take(3));

const mergeMap$ = source$.pipe(
    mergeMap(x => target$.pipe(colorize(x.color))) // colorize as source$ value
  );

const exhaustMap$ = source$.pipe(
    exhaustMap(x => target$.pipe(colorize(x.color)))
  );

const switchMap$ = source$.pipe(
    switchMap(x => target$.pipe(colorize(x.color)))
  );

const concatMap$ = source$.pipe(
    concatMap(x => target$.pipe(colorize(x.color)))
  );


// visualization
source$.subscribe(rxObserver('[source$] A stream that emits at [5ms, 10ms, 20ms]'));
target$
  .pipe(colorize('#ff5073'))
  .subscribe(rxObserver('[target$] will be mapped to a timer that emits at [N+0ms, N+3ms, N+6ms]'));
mergeMap$.subscribe(rxObserver('mergeMap'));
exhaustMap$.subscribe(rxObserver('exhaustMap'));
switchMap$.subscribe(rxObserver('switchMap'));
concatMap$.subscribe(rxObserver('concatMap'));


// helpers
function colorize(color) {
  return pipe(
    map(y => Marble(y, color))
  );
}

// creates a colored Marble
function Marble(value,color) {
  return {
    valueOf: ()=>value
    , color
  };
}

// like .from, but items are delayed by their value
function fromDelayed (arr) {
  return from(arr).pipe(
      delayWhen(x=>timer(x))
    );
}

```