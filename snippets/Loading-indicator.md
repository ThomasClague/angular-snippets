# Handling loading indicators in Angular Templates

There are two scenarios where handling loading indicators are useful in an Angular app

1. Initial loading of page data
2. Waiting for an action to complete


## Initial loading of page data
When working with observables in angular, it is best to try and use the Async Pipe to handle the subscribing and unsubscribing to observables 

To handle the loading indicator in a template while utilising the [Async Pipe](https://angular.io/api/common/AsyncPipe), you should do as follows

In the .ts file for your component
1.	Declare an observable at the component level
2. In OnInit, set your observable to the response from your service
```
public data$: Observable<string[]>;

constructor(private userService: UserService) {}

ngOnInit(): void {
  this.data$ = this.userService.getData();
}
```
In your template file
1. Create an ngIf that references your observable
2.  suffix the observable with the | async pipe 
3. use 'as varaibleName' to create a template reference variable to use within the context of your template 
	(this prevents multiple subscriptions in the template
4. use the else keyword to point to the loading template, this unifies your templates in a stateful manner
```
<div *ngIf="(data$ | async) as data; else loading">
  <div *ngFor="let text of data">
    <p>{{text}}</p>
  </div
</div>

<ng-template  #loading>
  <spinner></spinner>
</ng-template>
```

##  Waiting for an action to complete

To utilise the async pipe to trigger your loading indicator when you are waiting for an action to complete (submitting a form), you can use a subject which gets updated from an operator after the action has completed using a custom RxJs operator.

Create the operators
1. Create a new .ts file called operators.ts
2. Paste in the following code
```
// This operator invokes a callback upon subscription
export  function  prepare<T>(callback: () =>  void): (source: Observable<T>) =>  Observable<T> {
  return (source: Observable<T>): Observable<T> =>  defer(() => {
    callback();
    return  source;
  });
}

// This updates this subject upon subscription to the actual source stream via `indicator.next(true)`
export  function  indicate<T>(indicator: Subject<boolean>): (source: Observable<T>) => Observable<T> {
  return (source: Observable<T>): Observable<T> =>  source.pipe(
    prepare(() =>  indicator.next(true)),
    finalize(() =>  indicator.next(false))
  )
}
```

In the .ts for your component
1.	Declare an subject at the component level
2.	pipe your service call and call indicate(this.loading$)
3.	You must then subscribe to execute the submission method

```
loading$ = new  Subject<boolean>()

constructor(private userService: UserService) {}

create(): void {
  this.userService.submit(new User(name))
  .pipe(indicate(this.loading$))
  .subscribe();
}
```

In the template file
```
<button (click)="create()">Create User</button>
<div *ngIf="loading$ | async">
  <loading-indicator></loading-indicator>
</div>
  
```

## Things to remember
Never try to use both  methods at the same time
