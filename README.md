# Angular-RxJS
Find the associated Pluralsight course here: https://app.pluralsight.com/library/courses/rxjs-angular-reactive-development

`APM-Start`: The starter files for the course. **Use this to code along with the course**.

`APM-Final`: The completed files. Use this to see the completed solution from the course.

`APM-WithExtras`: The completed files with some extra code. Use this to see some additional techniques not included in the course.

I've developed a few additional examples, including using action streams to "pass" parameters and retrieving multiple related datasets here: https://stackblitz.com/edit/angular-todos-deborahk


## Awesome Course

Recommended.

## Extra Notes

### angularjs
```
ng build 
	same as 
	ng build —target=development —environment=dev
ng build —prod
	same as 
	ng build —prod -e=prod
```

### add new ajs with routing 
```
 ./node_modules/.bin/ng  new angrouting --routing -d

ng g c dashboard

ng g m admin --routing -d        # -d dry run adds admin module with routing

 ng g c admin

 ng g c admin/email-blast  # create sub component under admin

ng g guard auth -d
CREATE src/app/auth.guard.spec.ts (346 bytes)
CREATE src/app/auth.guard.ts (414 bytes)


npm install --save @angular/material @angular/animations
```
the animations are needed for menus etc
```
ng e2e 						#runs end to end tests in e2e folder
ng e2e --element-explorer		#runs e2e but allows debugging and shows debug web page - current issue 

ng g guard auth				#generate guard to protect an area of functionality
```

https://github.com/marley-nodejs/Creating-Apps-With-Angular-Node-and-Token-Authentication

fail build open shift rhmean
https://github.com/sass/node-sass/releases/download/v3.13.1/linux-x64-57_binding.node

https://github.com/sass/node-sass/releases/download/v4.9.0/linux-x64-57_binding.node

Immediately-Invoked Function Expression, or IIFE for short. It executes immediately after it’s created.
```
(function(){
    // all your code here
    var foo = function() {};
    window.onload = foo;
    // ...
})();
// foo is unreachable here (it’s undefined)
```

ui.router

is a better router - see plural sight angularjs line of business Applications
```
/**
 * This is PriceAnalyticsCtrl. It doesn’t use ‘.controller("PriceAnalyticsCtrl as vm“‘ syntax so the $scope is explicitly added  
 */
(function () {
    "use strict";

    angular
        .module("productManagement")
        .controller("PriceAnalyticsCtrl",
                    ["$scope",
                     "$filter",
                     "products",					// HERE products is injected from the def in app.js where the route to state assoc. is setup 
                     "productService",
                     PriceAnalyticsCtrl]);

    function PriceAnalyticsCtrl($scope,$filter, products, productService){
        $scope.title="Price Analytics";
	. . .
    }
}());
```


## Notes during ng-conf '19: Angular Blast-Off - Day 1

https://github.com/aaronfrost/angular-blastoff

vscode extension:

angular essentials

https://stackblitz.com/edit/angular-blastoff-rxjs1
https://stackblitz.com/edit/angular-blastoff-rxjs2

https://stackblitz.com/edit/angular-blastoff-rxjs4
```
import { of, from, interval } from 'rxjs';
import { map, filter, delay, share, take, tap, count, last, first, switchMap, catchError } from 'rxjs/operators';

console.clear();

const i = interval(100).pipe(
  //tap(console.log),
  filter(val=> val > 10),
  take(15),
  //count()
  switchMap(val=> interval(1000)),
  tap(val=> {
    throw new Error('Hello');
  }),
  catchError(error=> {
    console.warn(error);
    return of('FAILED');
  })
);

 const subscription = i.subscribe(
   val => console.log(1, val), 
   null, 
   ()=> console.log('completed')
);
```

with RxJS - use onPush change detection strategy - performance boost

https://app.pluralsight.com/paths/conference/ng-conf-2019
whole bunch of courses!


https://app.pluralsight.com/player?course=angular-denver-2019-session-35&author=angular-denver&name=875a3c0a-bdbc-4b8c-9a42-12f543635571&clip=0&mode=live
```
const obs = new Observable(
  (subscriber) => {
    let i = 0;
    const iID = setInterval(
      () => subscriber.next(i++),
      1000
    );
    const td = () => {
      clearInterval(iID);
    };
    return td; // called on unsubscribe, error or complete
  }
);
const subs = obs.subscribe(
     val => console.log(11, val), 
   null, 
   ()=> console.log('completed')
);
```

An observable has producer inside it.  —> unicast

A subject has the producer outside.    —> multicast

Both have subscriptions.

#### Subject types: Subject, BehaviorSubject ReplaySubject, AsyncSubject, WebSocketSubject

Subject - forward to multiple subscribers at the same time - starts as soon as there is ONE subscriber?!
If you re-subscribe to an already completed subject - nothing happens
If you re-subscribe to an already error subject - you receive the error from the PAST

BehaviorSubject needs default value. emits current value and future values to all subscribers
use case - lightweight state managment

ReplaySubject - will replay past as well as future values
If re-subscribe to an already completed subject - receive all items and then complete event
cache has FIFO principle
1st param buffer size, 2nd param windowTime (lifetime of cache() emptied after this time 
use cases - history of notifications, timing and late subscribers

AsyncSubject
has cache of values, on complete the latest value is sent to subscribers
If subscribe AFTER completion - STILL receive latest value and then complete event

dev.to/rxjs

Task stack - synchronously (mIcro task stack ; mAcro task stack)
```
const s$ = from ([1,2]);
const sub = s$.subscribe(console.log);
sub.unsubscribe();
```

This shows NO output - because events are scheduled to mAcro task AND unsubscribe happens before that time
```
const s$ = from ([1,2]).pipe(observeOn(asyncScheduler));
const sub = s$.subscribe(console.log);
sub.unsubscribe();
```

https://github.com/DeborahK/Angular-RxJS

### 9 - BehaviorSubject & Subject are referred to as actionable streams
/Users/robinjohnhopkins/workspace/rxjs-angular-reactive-development/08/demos/APM-m8/src/app/products/product-

list.component.ts
```
// create actionable stream with default value
private categorySelectedSubject = new BehaviorSubject<number>(0);
categorySelectedAction$ = this.categorySelectedSubject.asObservable();

// combine streams
  products$ = combineLatest([
    this.productService.productsWithCategory$,
    this.categorySelectedAction$
  ])
    .pipe(
      map(([products, selectedCategoryId]) =>
        products.filter(product =>
          selectedCategoryId ? product.categoryId === selectedCategoryId : true
        )),
      catchError(err => {
        this.errorMessage = err;
        return EMPTY;
      })
    );

  categories$ = this.productCategoryService.productCategories$
    .pipe(
      catchError(err => {
        this.errorMessage = err;
        return EMPTY;
      })
    );
  // change actionable stream on user action
  onSelected(categoryId: string): void {
    this.categorySelectedSubject.next(+categoryId);
  }
```

### 08/demos/APM-m8/src/app/products/product-list.component.html
```
// use streams in angular html template
<div class="card">
  <div class="card-header">
    {{pageTitle}}
  </div>

  <div class="card-body">
    <div class="container">
      <div class="row justify-content-between">
        <div class="col-3">
          <select class="form-control"
                  (change)="onSelected($event.target.value)">
            <option value="0">- Display All -</option>
            <option *ngFor="let category of categories$ | async"
                    [value]="category.id">{{ category.name }}</option>
          </select>
        </div>
        <div class="col-2">
          <button type="button"
                  class="btn btn-outline-secondary"
                  (click)="onAdd()">Add Product</button>
        </div>
      </div>
    </div>

    <div class="table-responsive">
      <table class="table mb-0"
             *ngIf="products$ | async as products">
        <thead>
          <tr>
            <th>Product</th>
            <th>Code</th>
            <th>Category</th>
            <th>Price</th>
            <th>In Stock</th>
          </tr>
        </thead>
        <tbody *ngFor="let product of products">
          <tr>
            <td>{{ product.productName }}</td>
            <td>{{ product.productCode }}</td>
            <td>{{ product.category }}</td>
            <td>{{ product.price | currency:"USD":"symbol":"1.2-2" }}</td>
            <td>{{ product.quantityInStock }}</td>
          </tr>
        </tbody>
      </table>
    </div>

  </div>
</div>

<div class="alert alert-danger"
     *ngIf="errorMessage">
  {{ errorMessage }}
</div>
```

### 10 - ReplaySubject - will replay past as well as future values - cache HttpClient GETs
~/workspace/rxjs-angular-reactive-development/10/demos/APM-m10/src/app/product-categories/product-category.service.ts
```
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { throwError } from 'rxjs';
import { ProductCategory } from './product-category';
import { tap, catchError, shareReplay } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class ProductCategoryService {
  private productCategoriesUrl = 'api/productCategories';

  productCategories$ = this.http.get<ProductCategory[]>(this.productCategoriesUrl)
    .pipe(
      tap(data => console.log('categories', JSON.stringify(data))),
      shareReplay(1),				// <<<<<<<<<<<<<<<< call get ONCE and cache it
      catchError(this.handleError)
    );

  constructor(private http: HttpClient) { }

  private handleError(err: any) {
    // in a real world app, we may send the server to some remote logging infrastructure
    // instead of just logging it to the console
    let errorMessage: string;
    if (err.error instanceof ErrorEvent) {
      // A client-side or network error occurred. Handle it accordingly.
      errorMessage = `An error occurred: ${err.error.message}`;
    } else {
      // The backend returned an unsuccessful response code.
      // The response body may contain clues as to what went wrong,
      errorMessage = `Backend returned code ${err.status}: ${err.body.error}`;
    }
    console.error(err);
    return throwError(errorMessage);
  }
}
```

Docs: https://rxjs-dev.firebaseapp.com/api/operators/shareReplay
```
shareReplay<T>(configOrBufferSize?: number | ShareReplayConfig, windowTime?: number, scheduler?: SchedulerLike): MonoTypeOperatorFunction<T>

Parameters
configOrBufferSize	Optional. Default is undefined.
Type: number | ShareReplayConfig.
windowTime	Optional. Default is undefined.
Type: number. ms
scheduler	Optional. Default is undefined.
Type: SchedulerLike.
```

### 11 - Higher Order Observables - take observable and emit observable
#### concatMap; mergeMap; switchMap

concatMap - wait for prior observable to complete before starting the next one
```
  suppliersWithConcatMap$ = of(1, 5, 8)
    .pipe(
      tap(id => console.log('concatMap source Observable', id)),
      concatMap(id => this.http.get<Supplier>(`${this.suppliersUrl}/${id}`))
    ); // will return a stream that has three entries IN ORDER
```

mergeMap - execute inner observables in PARALLEL then merges - AKA flatmap
```
  suppliersWithMergeMap$ = of(1, 5, 8)
    .pipe(
      tap(id => console.log('mergeMap source Observable', id)),
      mergeMap(id => this.http.get<Supplier>(`${this.suppliersUrl}/${id}`))
    );// will return a stream that has three entries UNORDERED but inner calls are ASYNC
```

switchMap - stop any prior observable before switching to the next one
e.g. type ahead; user selection from a list
```
  suppliersWithSwitchMap$ = of(1, 5, 8)
    .pipe(
      tap(id => console.log('switchMap source Observable', id)),
      switchMap(id => this.http.get<Supplier>(`${this.suppliersUrl}/${id}`))
    ); // will return a stream that has ONE entry of most recently projected Observable i.e. 8 return
```

￼```
supplierWithMergeMap$ = of(1, 5, 8) 
		.pipe( 
		mergeMap(supplierId => this.http.get<Supplier>(`${this.url}/${supplierId}`)) 
	); 
selectedProductSuppliers$ = this.selectedProduct$ .pipe( 
	mergeMap(product => 
		from(product.supplierIds) 
			.pipe(mergeMap(supplierId => this.http.get<Supplier>(`${this.url}/${supplierId}`)),
				toArray() 
))); 

``

service.ts
NB: The inner construct has from which always completes AND toArray which always needs completion.

~/workspace/rxjs-angular-reactive-development/12/demos/APM-m12/src/app/products/product.
```
### service.ts
  selectedProductSuppliers$ = this.selectedProduct$
    .pipe(
      filter(selectedProduct => Boolean(selectedProduct)),
      switchMap(selectedProduct =>
        from(selectedProduct.supplierIds)
          .pipe(
            mergeMap(supplierId => this.http.get<Supplier>(`${this.suppliersUrl}/${supplierId}`)),
            toArray(),
            tap(suppliers => console.log('product suppliers', JSON.stringify(suppliers)))
          )));
```          
NB: Note trick to skip process if  selectedProduct is undefined or null - filter

NB: Note switchMap which only returns for last selected product


### Combine All Streams example
~/workspace/rxjs-angular-reactive-development/12/demos/APM-m12/src/app/products/product-list-alt/product-detail.component.ts
```
  vm$ = combineLatest([							    // combine 3 streams to one
    this.product$,
    this.productSuppliers$,
    this.pageTitle$
  ])
    .pipe(
      filter(([product]) => Boolean(product)),			     // null undefined trick
      map(([product, productSuppliers, pageTitle]) =>
        ({ product, productSuppliers, pageTitle }))                  // convert array to structure
    );
```

### 12/demos/APM-m12/src/app/products/product-list-alt/product-detail.component.html
```
<div class="card"
     *ngIf="vm$ | async as vm">
  <div class="card-header">
    {{vm.pageTitle}}
  </div>

  <div class="card-body">

    <div class="row">
      <div class="col-md-2">Name:</div>
      <div class="col-md-6">{{vm.product.productName}}</div>
    </div>
```

### To Http Post to Add Item
Check out the APM-WithExtras folder in github repo for this course. It provides code to handle that. Here is a bit of it:
```
  // Action Stream for adding products to the Observable
  private productInsertedSubject = new Subject<product>();
  productInsertedAction$ = this.productInsertedSubject.asObservable();

  // Add the newly added product via http post with concatMap
  // And then to the full list of products with scan.
  headers = new HttpHeaders({ 'Content-Type': 'application/json' });
  productsWithAdd$ = merge(
    this.productsWithCategory$,
    this.productInsertedAction$
      .pipe(
        concatMap(newProduct => {
          newProduct.id = null;
          return this.http.post<product>(this.productsUrl, newProduct, { headers: this.headers })
            .pipe(
              tap(product => console.log('Created product', JSON.stringify(product))),
              catchError(this.handleError)
            );
        }),
      ))
    .pipe(
      scan((acc: Product[], value: Product) => [...acc, value]),
      shareReplay(1)
    );
```

#### The scan operator is great to use anytime you want to retain an "accumulated state"

See delete scan example here
I have a CRUD example in the APM-WithExtras folder in the github repo for this course.
I applied those techniques to your code and you can find the result here: https://stackblitz.com/edit...


### Code for a Refresh feature is provided in the APM-WithExtras folder:
```
  // To support a refresh feature
  private refresh = new BehaviorSubject<boolean>(true);

  products$ = this.refresh
    .pipe(
      mergeMap(() => this.http.get<product[]>(this.productsUrl)
        .pipe(
          tap(data => console.log('Products', JSON.stringify(data))),
          catchError(this.handleError)
        )
      ));
```
