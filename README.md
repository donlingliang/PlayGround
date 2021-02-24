# Playground
Everything starts from a question posted on [Stackoverflow](https://stackoverflow.com/questions/58241972/flatmap-does-not-work-v-s-foreach-in-rxjava), I'm trying to figure out why RxJava code snippet **doesn't work** in that question, then people's answers blew my Rx mind model totally.

In this repository, I am taking notes on my own experimental code/articles/videos, so that I could rebuild the correct mindset on Rx before I unleash this powerful tool.

**Doesn't work case:**
```
Observable.just(getListNetworkCall())
		.flatMapIterable(response -> response.getDataList())
		.flatMap(dataType -> syncData(dataType))
		.subscribeOn(Schedulers.io())
		.subscribe(new Consumer<Response2>() {
			@Override
			public void accept(Response2 response) throws Throwable {
				System.out.println("response:" + response.toString());
			}}
		);

private Observable<Response2> syncData(String dataType) {
        return Observable.just(syncDataNetworkCall(dataType));
}

private Response2 syncDataNetworkCall(String dataType) {  
		System.out.println("dataType:" + dataType + " " + Thread.currentThread().getName().toString() + "::id=" + Thread.currentThread().getId());  
		return this.getDefinition(dataType);  
}
```
Assuming we want to do a `getListNetworkCall()` for a list of dataType, then do `syncDataNetworkCall()` on each dataType, using **flatmap** to map dataType to Response2, flatten them into one observable and print out the responses. This looks fine but actually it doesn't work ---> **nothing is printed out**.  Based on [Sanlok Lee's answer](https://stackoverflow.com/questions/58241972/flatmap-does-not-work-v-s-foreach-in-rxjava), and [further reading](https://proandroiddev.com/understanding-rxjava-subscribeon-and-observeon-744b0c6a41ea) we know that Main function(Calling thread) has finished before the IO thread fires/returns anything.

If I put the Thread.sleep() for 3 seconds with the `subscribeOn(Schedulers.io())`, `syncData()` will run on *RxCachedThreadScheduler-1* which is the io thread I initially wanted it to run on. 

However, if I removed the `subscribeOn(Schedulers.io())` and put it in the `syncData` instead, like this
```
Observable.just(first network call)
		.flatMapIterable(...)
		.flatMap(dataType -> syncData(dataType))
		.subscribe(...);

private Observable<Response2> syncData(String dataType) {
        return Observable.just(syncDataNetworkCall(dataType)).subscribeOn(Schedulers.io());
}

private Response2 syncDataNetworkCall(String dataType) {
	System.out.println("dataType:" + dataType + " " +
	Thread.currentThread().getName().toString() + "::id=" + Thread.currentThread().getId()); 
	return this.getDefinition(dataType); 
}
```
It prints *Main* as the Thread name still, so it means that `subscribeOn()`may only take care of the stream when it start emitting instead from the calling thread.

Reading Lists:
- [Advanced Reactive Java](http://akarnokd.blogspot.com/)
- [Thread Model](https://colobu.com/2016/07/25/understanding-rxjava-thread-model/)

**Rx in Server**

Reading Lists:
- [Why we use Rx in Server](https://stackoverflow.com/questions/49226946/is-rx-java-something-a-server-side-engineer-needs)
- [Reactive Programming in the Netflix API with RxJava](https://netflixtechblog.com/reactive-programming-in-the-netflix-api-with-rxjava-7811c3a1496a)
- [Reactive programming with Jersey](https://sleeplessinslc.blogspot.com/2015/04/reactive-programming-with-jersey-and.html)

> Written with [StackEdit](https://stackedit.io/).
