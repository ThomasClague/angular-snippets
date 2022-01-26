# Handling loading indicators in Angular Templates

There are two scenarios where handling loading indicators are useful in an Angular app

1. Initial loading of page data
2. Waiting for an action to complete


## Initial loading of page data
When working with observables in angular, it is best to try and use the Async Pipe to handle the subscribing and unsubscribing to observables 

To handle the loading indicator in a template while utilising the [Async Pipe](https://angular.io/api/common/AsyncPipe), you should do as follows

In your .ts file
1.	Declare an observable at the component level
2. In OnInit, set your observable to the response from your service
```
public data$: Observable<string[]>;

ngOnInit(): void {
  this.data$ = this._service.getData();
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
