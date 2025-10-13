---
date: "2025-10-12T15:30:00+01:00"
draft: false
title: "Spring AOP - A tiny introduction"
tags: ["Spring AOP", "AOP"]
---
**Hey there, reader! 👋**

I hope you’re feeling good today, because we’re about to dive into Spring AOP — which, at first, might not mean much.

I first came across it at work when I was trying to use the _**@Transactional**_ and _**@Retryable**_ annotations in Spring, and for some reason… they just weren’t working. So — like any other responsible developer — I rushed to ChatGPT and begged it to explain why the hell those annotations, which worked flawlessly elsewhere, refused to work here.

After a friendly back-and-forth, it told me: _“That’s because of how Spring AOP works.”_

And that was the first time I’d ever heard about **AOP**.  
Bear in mind that the last time I’d touched Java was back in university — quite a while ago — and I didn’t have a solid Spring background when I started this job. So I’m still piecing things together.

If you’ve stumbled across this post, you’ve probably heard “AOP” or “Spring AOP” somewhere — maybe even from ChatGPT — while debugging something mysterious in your code.

Either way, you’re here, I’m here, so let’s figure this out together. Because honestly, I’m learning as I write this too!
# AOP? What doest that mean and what is it?

AOP. At first glance, this acronym doesn’t say much.  
You can kind of guess that the “O” stands for _Object_ and the “P” for _Programming_ or _Paradigm_ — at least that was my first guess.  
But ChatGPT (and a great post from [Baeldung](https://www.baeldung.com/spring-aop)) cleared things up for me.

**AOP** stands for **Aspect-Oriented Programming**.  
It’s a programming paradigm that allows us to add new behavior to our existing code **without modifying the code itself** — pretty cool, right?

You might be thinking: _can’t we already do that with SOLID principles and design patterns like the Decorator or Proxy?_  
And yes — you’re right.  
Principles like the **Open/Closed Principle** and patterns such as **Decorator** or **Wrapper** let us extend our system by adding **domain-specific behaviors** while keeping things modular.

But **AOP** targets a different kind of behavior — one that _cuts across_ layers or classes.  
It’s meant for code that doesn’t belong to a single module but rather affects multiple parts of the system.  
These are known as **cross-cutting concerns**.
## Cross-cutting concerns?

If you’re not yet familiar with this concept (I had very little experience with it too), here’s what I learned.

**Cross-cutting concerns** are functionalities that are needed in many places: **logging**, **error handling**, **security/authorization**, **monitoring**, and so on.

Take logging as an example.  
We need logs across our application — in services, processes, HTTP requests, etc.  
Since logging is needed _everywhere_, it “cuts across” many parts of the system.

If we tried to place this functionality directly inside each module (Users, Orders, Payments…), we’d quickly end up duplicating code.  
AOP gives us a way to keep our modules clean 🧹 while still applying these shared behaviors wherever they’re needed.
### Let's see some examples - Logging
Let’s imagine a simple `PaymentService` — nothing fancy.
##### Without AOP
```
public class PaymentService {

    public void processPayment() {
        Logger.info("Starting processPayment...");
        if(!SecurityContext.hasPermission("PAYMENT")) {
            throw new SecurityException("No permission!");
        }
        // business logic here
        Logger.info("Finished processPayment");
    }
}
```
Without AOP, if we want to log before and after our method, we have to explicitly add those lines — and we’d need to do the same across multiple services.
##### With AOP
```
@Aspect
@Component
public class LoggingAspect {

    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}

    @Before("serviceMethods()")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println(">>> Starting " + joinPoint.getSignature().getName());
    }

    @After("serviceMethods()")
    public void logAfter(JoinPoint joinPoint) {
        System.out.println("<<< Finished " + joinPoint.getSignature().getName());
    }
}
```

```
public class PaymentService {

    public void processPayment() {
        if(!SecurityContext.hasPermission("PAYMENT")) {
            throw new SecurityException("No permission!");
        }
        // business logic here
    }
}
```
Now our `PaymentService` contains **only business logic**, while logging is automatically applied across all service methods — no duplicated code, no clutter.

Don’t worry about that suspicious-looking `LoggingAspect` class — we’ll talk about it in the next section.
# Core concepts
### Understanding the building blocks of AOP

Before diving into example code, let’s first understand _what_ we actually need to implement this paradigm in Spring — and I must say, at first glance, these concepts are not very intuitive.

In Spring AOP, we work with four key components:
- **Aspect**
- **Join Point**
- **Advice**
- **Pointcut**

If you’re anything like me, looking at that list the first time probably left you thinking:  
_What on earth is an Aspect? Advice for what? Join Point? Pointcut? Does one join things and the other cut them apart?_ 😅

So yes — these terms raised a lot of questions for me too. Maybe it’s because English isn’t my first language, or maybe it’s just because these names sound unnecessarily mysterious.  
But after reading the official docs and, of course, asking my friend ChatGPT, I finally started to get it.
### Aspect
An **Aspect** is the module that implements a _cross-cutting concern_.  
It’s basically a class where you define the code you want to execute across multiple points of your application.  
Pretty simple, right? Once you understand that, most of the rest builds around it.
### Join Point and Pointcut

These two were the hardest for me to grasp.

A **Join Point**, in general AOP terms, is _a moment where something happens_ — for example:
- a method call or execution,
- an object instantiation,
- a constructor call,
- an exception being handled, etc.

However, in **Spring AOP**, Join Points are _limited_ to **method executions**.  
And not just any methods — only those that belong to **Spring-managed beans** and are **eligible for proxying**.  
That means AOP only kicks in when methods are called _through Spring’s dependency injection system_ (more on that later).

Now, the **Pointcut** defines _which_ Join Points should trigger your cross-cutting code.  
It’s basically a matching expression — you tell Spring, _“run my cross-cutting logic for all methods that match this pattern.”_

So:
- A **Join Point** is a potential point where your cross-cutting code _could_ run.    
- A **Pointcut** is a filter that decides _which_ of those Join Points your Aspect should actually apply to.
### Advice

Man, the naming here is wild.

**Advice** refers to _what_ you actually want to do _at_ those Join Points matched by a Pointcut.  
It’s the **action** — the code that runs before, after, or around the target method.  
You define it inside your Aspect class and use annotations to specify _when_ it should execute.

There are several types of advice in Spring AOP:
- **`@Before`** – Runs **before** the join point (method execution).
- **`@After`** – Runs **after** the join point, **regardless** of whether it completes normally or throws an exception (like a `finally` block).
- **`@AfterReturning`** – Runs **after** the method **returns successfully**.
- **`@AfterThrowing`** – Runs **only if** the method **throws an exception**.
- **`@Around`** – The **most powerful** type of advice.  
    It _wraps_ the method execution, giving you full control over:
    - Whether the method should run at all,
    - When it should run,
    - What value to return,
    - Or whether to throw an exception instead.
# Now *PointCutting* it all together
Let’s take everything we’ve learned and see how it looks in a practical, visual example — a definition of a cross-cutting functionality.
```
@Aspect
@Component
public class LoggingAspect {

    // ----- POINTCUT -----
    @Pointcut("execution(* com.example.service.OrderService.*(..))")
    public void orderServiceMethods() {} 

    // ----- ADVICE (Before) -----
    @Before("orderServiceMethods()")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println(">>> Before method: " + joinPoint.getSignature().getName());
    }

    // ----- ADVICE (AfterReturning) -----
    @AfterReturning(pointcut = "execution(* com.example.service.OrderService.placeOrder(..))", returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("<<< After method: " + joinPoint.getSignature().getName() + " | Returned: " + result);
    }
}
```
In this example, our class defines a cross-cutting functionality — so it’s an **Aspect** — and it’s marked with the `@Aspect` annotation.

We start with a **Pointcut**, defined using the `@Pointcut` annotation on an empty method.  
Why an empty method? Because we need a way to _name_ our Pointcut expression — something we can reference later — but we don’t want to execute it.  
That empty method is never called at runtime; it simply serves as a label for the Pointcut.

Next, we have our **advices**, the actual cross-cutting methods we want to run.
- The `@Before` advice references the `orderServiceMethods()` Pointcut we just defined, telling Spring to execute `logBefore()` **before every method** in `OrderService`.  
    Pretty straightforward.
- The `@AfterReturning` advice is a bit different. It uses two attributes:
    - **`pointcut`**, where we define a _more specific expression_ (in this case, only the `placeOrder()` method).
    - **`returning`**, which tells Spring to capture the method’s return value and inject it into the `result` parameter of our advice.

We could have reused the same Pointcut method, but this shows how to target a specific method for finer control.

So when `placeOrder()` returns something like `"Order confirmed"`, Spring passes that value into `result`, and our advice logs it.

You might be wondering: _“Okay, I get what an Aspect is now. But what does this have to do with the problem you had earlier with Spring annotations?”_

Well, the connection is that those annotations — like `@Transactional` and `@Retryable` — are actually **Aspects inside the Spring Framework**.  
So, to understand why they sometimes don’t behave as expected, we first need to understand how AOP and these concepts work behind the scenes.
# I got all of this, but… how are they executed?

Now that we’ve learned what AOP and Spring AOP are — and the core concepts behind them — the next question that naturally pops up is:  
_How does my Aspect actually get “injected” or “triggered”?_

The answer lies in the process called **Weaving**.

If you’re not familiar with the word _weaving_ (I wasn’t either at first), it literally means _the process of interlacing two sets of threads or yarns at right angles to create a fabric._

Using that metaphor, you can think of it like this:
- One thread is your **Aspect** (the cross-cutting code),
- The other thread is your **application logic** (the matched methods / Join Points).

When these two are interlaced, you get the final “fabric” — your complete application.  
So, **weaving** is the process of linking Aspects (from Spring or from your own code) with the rest of your application logic.
### Types of Weaving
There are a few different types of weaving depending on _when_ the Aspects are applied:
#### **Compile-time weaving**
- Aspects are combined with the source code **during compilation**.
- The resulting `.class` files already contain the woven aspect logic.
#### **Load-time weaving (LTW)**
- Aspects are combined **when classes are being loaded by the JVM’s class loader.**
- This happens dynamically during class loading, using a _weaving agent_ such as **AspectJ**.
- Spring can integrate with AspectJ for this, but it’s less common in typical Spring Boot apps.
#### **Runtime weaving (proxy-based)**
- This is what **Spring AOP** uses.
- Aspects are applied **at runtime** by creating **proxies** that wrap your beans.
- When a method is called, the proxy intercepts the call, executes your advice, and then delegates to the real method.    
- Because it’s done dynamically, **it doesn’t modify bytecode** — it simply adds an extra layer at runtime.
# Understanding the issue I was having

Remember the issue I mentioned at the beginning of the post?  
Well, it turns out that the culprit — besides me not fully understanding how things work — was this **proxy-based approach** that Spring AOP uses.  
So let’s dive into how this proxy works and why a small detail can have a _big_ impact on your code.
 
An **AOP proxy** is an object created by the AOP framework that’s responsible for applying the behavior defined in aspects — in other words, executing the **advice** methods associated with them.

When Spring creates a bean, it checks whether any aspect targets that bean.  
If so, instead of returning the original bean directly, Spring creates a **proxy object** that wraps the original bean instance.

This proxy is responsible for:
- **Intercepting** method calls made on the bean,
- **Running** all advice logic defined in aspects,
- **Delegating** the call to the actual target method afterward.

Conceptually:
**Normal class call**
```
Client
   │
   ▼
Real Bean (PaymentService)
   │
   ▼
processPayment()
```

**Spring AOP proxy call**
```
Client
   │
   ▼
AOP Proxy (created by Spring)
   │   ▲
   │   └───── Intercepts call
   │          Executes all matching advices
   │          (e.g. @Before, @After, @Around)
   ▼
Real Bean (PaymentService)
   │
   ▼
processPayment()

```

This mechanism is what we call **runtime weaving** in Spring AOP — it integrates the aspect behavior dynamically through proxies, without modifying bytecode.

At this point, I thought: _“Okay, I get it now… but why wasn’t it working?”_

Well, here’s the missing piece:  
I was using the right annotations (`@Transactional`, `@Retryable`), but I was calling those annotated methods **from within the same class** — from another method of that same bean.

And that’s exactly where things broke.
### The Self-Invocation Problem

The Spring documentation explains that when a bean is proxied, **calls must go through the proxy** for AOP to work.  
If you call a method from inside the same class (via `this`), the proxy isn’t involved — you’re calling the real bean directly.

So the aspect never has a chance to run.  
In other words, you _bypass_ the proxy completely.

> _“...self invocation is not going to result in the advice associated with a method invocation getting a chance to run. In other words, self invocation via an explicit or implicit `this` reference will bypass the advice.”_  
> — _Spring Framework Documentation_

As a result, self-invocation will **not trigger** aspects such as `@Transactional`, `@Retryable`, or even your own custom aspects.
### How to Solve It

You have a few options:
1. **Avoid self-invocation (recommended)**
    - The approach recommended by the Spring documentation is to **refactor your code** so this situation doesn’t occur.  
    - In simple terms, call the annotated method from _another class_ that has the original class injected as a dependency.  This way, the call goes through the proxy.
2. **Self-injection (alternative)**
	- This consists of injecting the class into itself.  
	- That way, the injected reference passes through the proxying process, and any aspect (like `@Transactional` or `@Retryable`) is correctly applied.
    - Example:
    ```
      @Component
		public class MyService {

		    @Autowired
		    private MyService self; // proxied reference
		
		    public void methodA() {
		        self.methodB(); // goes through proxy
		    }
		
		    @Transactional
		    public void methodB() {
		        // transactional logic
		    }
	    }
    ```
3. **Using `AopContext.currentProxy()` (last resort)**
	- As per the documentation, this approach is **strongly discouraged** and should only be used as a _last resort_ solution.
	- Example (from the official docs):
	  ```
	  public class SimplePojo implements Pojo {

	    public void foo() {
        // Works, but should be avoided if possible
        ((Pojo) AopContext.currentProxy()).bar();
		    }
		
		    public void bar() {
		        // some logic...
		    }
		}
	    ```  

You can read more about proxies [here](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html#aop-understanding-aop-proxies).
# See you in the next post... 👋

We finally reached the end — woof.

I hope this post was as beneficial to you as it was for me.

I decided to write this post for three reasons:
- I find it easier to learn new things when I have to research them and try to explain them to someone else — instead of just reading the same text over and over again until it burns into my brain.
- I wanted a way to document my learnings instead of relying solely on my shrinking, downgrading “internal storage device.”  
    _Weren’t books invented in the first place because our tiny brains couldn’t store everything we wanted — and occasionally had to “wipe out” old data to make space for new things?_
- At work, I’ve been dealing with new kinds of problems and situations (like this one), so if I can pass along that knowledge, great — and if not, well, it still helped me learn.
- Maybe my way of explaining things helps someone out there. Or maybe not. But it is what it is, I guess.

I already have a few more topics I’d like to cover — see you in the next one!
### References

- [Spring AOP: Introduction](https://docs.spring.io/spring-framework/reference/core/aop/introduction-defn.html)
- [Spring AOP: Proxies](https://docs.spring.io/spring-framework/reference/core/aop/introduction-proxies.html)
- [Understanding AOP Proxies](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html#aop-understanding-aop-proxies)
- [Self-Injection with `@Autowired`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html#beans-autowired-annotation-self-injection)
- [Baeldung: Spring AOP Overview](https://www.baeldung.com/spring-aop)
- [Baeldung: AspectJ](https://www.baeldung.com/aspectj)
- [Baeldung: Spring AOP Annotations](https://www.baeldung.com/spring-aop-annotation)
