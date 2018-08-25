---
title: '[NG] 考古 - HttpInterceptor 循环引用错误'
date: 2018-08-25 09:13:48
tags: [Angular]
---

## 前言

恍然间发现这个错误已经不复存在了，于是稍微看了下相关 issue、commit、PR。写篇笔记祭奠下～

## 需求描述

一个使用 `HttpInterceptor` 的常见场景是实现基于 token 的验证机制。

为什么要使用拦截（intercepting）呢？

因为，在基于 token 的验证机制中，证明用户身份的 token 需要被附带在每一个（需要验证的请求的）请求头。如果不使用拦截手段，那么（由 `HttpClient` 实例触发的）每一个请求都需要手动修改请求头（header）。显然手动修改是繁琐和难以维护的。所以，我们选择做拦截。

[Angular 官网](https://angular.io/guide/http#set-default-headers)也给出了范例，以下代码可以实现一个 `AuthInterceptor` 拦截器：

```typescript
import { Injectable } from '@angular/core';
import { HttpEvent, HttpHandler, HttpInterceptor, HttpRequest } from '@angular/common/http';
import { Observable } from 'rxjs/Observable';
import { AuthService } from '../auth.service';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {

  constructor(private auth: AuthService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const authToken = this.auth.getAuthorizationToken();

    const authReq = req.clone({
      headers: req.headers.set('Authorization', authToken)
    });

    return next.handle(authReq);
  }
}
```

## 问题描述

但在 [5.2.3](https://github.com/angular/angular/blob/master/CHANGELOG.md#523-2018-01-31) 之前，执行上述官方给出的代码是会报错的。原因是 **存在循环引用问题**！

<!-- more -->

### 依赖关系1

我们看一下上述代码：`AuthInterceptor` 由于需要使用 `AuthService` 服务提供的获取 token 的方法，依赖注入了 `AuthService`：

```
AuthInterceptor -> AuthService  // AuthInterceptor 拦截器需要 AuthService 服务来获取 token
```

### 依赖关系2

而一般情况下我们的 `AuthService` 需要做登录登出等操作，特别是需要和后端交互以获取 token，所以需要依赖注入 `HttpClient`，存在依赖关系：

```
AuthService -> HttpClient // AuthService 服务需要 HttpClient 服务来和后端交互
```

### 依赖关系3

从下述源码可以看出，`HttpClient` 服务依赖注入了 `HttpHandler`：

```typescript
// v5.2.x
export class HttpClient {
  constructor(private handler: HttpHandler) {}

  request(...): Observable<any> {
    let req: HttpRequest<any>;
    ...
    // Start with an Observable.of() the initial request, and run the handler (which
    // includes all interceptors) inside a concatMap(). This way, the handler runs
    // inside an Observable chain, which causes interceptors to be re-run on every
    // subscription (this also makes retries re-run the handler, including interceptors).
    const events$: Observable<HttpEvent<any>> =
        concatMap.call(of (req), (req: HttpRequest<any>) => this.handler.handle(req));
    ...
}
```

而 `HttpHandler` 的依赖中包含可选的 `new Inject(HTTP_INTERCEPTORS)`：

```typescript
// v5.2.2
@NgModule({
  imports: [...],
  providers: [
    HttpClient,
    // HttpHandler is the backend + interceptors and is constructed
    // using the interceptingHandler factory function.
    {
      provide: HttpHandler,
      useFactory: interceptingHandler,
      deps: [HttpBackend, [new Optional(), new Inject(HTTP_INTERCEPTORS)]],
    },
    HttpXhrBackend,
    {provide: HttpBackend, useExisting: HttpXhrBackend},
    ...
  ],
})
export class HttpClientModule {
}
```

其中，`HTTP_INTERCEPTORS` 是一个 `InjectionToken` 实例，用于标识所有拦截器服务。`new Inject(HTTP_INTERCEPTORS)` 可以获取拦截器服务的实例们。

> 这里的“token”是 Angular 的 DI 系统中用于标识以来对象的东西。token 可以是字符串或者 `Type`/`InjectionToken`/`OpaqueToken` 类的实例。
>
> P.S. 关于使用哪一种 token 更好的问题，可以【TODO:】看一下[这篇文章](https://blog.thoughtram.io/angular/2016/05/23/opaque-tokens-in-angular-2.html)（[译文](https://segmentfault.com/a/1190000008626348)）。

也就是说，`HttpClient` 依赖于所有 `HttpInterceptor`s，包括 `AuthInterceptor`：

```
HttpClient -> AuthInterceptor // HttpClient 服务需要 AuthInterceptor 在内的所有拦截器服务来处理请求
```

### 循环依赖

综上，我们有循环依赖：

```
AuthInterceptor -> AuthService -> HttpClient -> AuthInterceptor -> ...
```

而在 Angular 里，每一个服务实例的初始化所需要的依赖都是需要事先准备好的，但一个循环依赖是永远也准备不好的……Angular 因此会检测循环依赖的存在，并在循环依赖被检测到时报错，部分源码如下：

```typescript
// v5.2.x
export class NgModuleProviderAnalyzer {
  private _transformedProviders = new Map<any, ProviderAst>();
  private _seenProviders = new Map<any, boolean>();
  private _allProviders: Map<any, ProviderAst>;
  private _errors: ProviderError[] = [];

  ...

  private _getOrCreateLocalProvider(token: CompileTokenMetadata, eager: boolean): ProviderAst|null {
    const resolvedProvider = this._allProviders.get(tokenReference(token));
    if (!resolvedProvider) {
      return null;
    }
    let transformedProviderAst = this._transformedProviders.get(tokenReference(token));
    if (transformedProviderAst) {
      return transformedProviderAst;
    }
    if (this._seenProviders.get(tokenReference(token)) != null) {
      this._errors.push(
        new ProviderError(`Cannot instantiate cyclic dependency! ${tokenName(token)}`, resolvedProvider.sourceSpan));
      return null;
    }
    this._seenProviders.set(tokenReference(token), true);
    ...
  }
}
```

让我们稍微看一下代码：

- `NgModuleProviderAnalyzer` 内部通过 `Map` 类型的 `_seenProviders` 来记录看到过的供应商。
- 在其方法 `_getOrCreateLocalProvider` 内部判断是否已经看过，如果已经看过会在 `_errors` 中记录一个 `ProviderError` 错误。

我用 5.2.2 版本的 Angular 编写了[一个遵循官方文档写法但出现“循环引用错误”的示例项目](https://github.com/baishusama/fe-grocery-store/tree/master/ng-5.2.2-cyclic-dep-err-with-interceptor)。下面是我 `ng serve` 运行该应用后，在 `compiler.js` 中添加断点调试得到的结果：

- 图一、截图时 `_seenProviders` 中已经记录的各个供应商：
![_seenProviders](https://user-images.githubusercontent.com/9972503/44579887-87e58b00-a7ca-11e8-9c32-ca2dfbc18f69.png)
- 图二、截图时 `token` 变量的值：
![token](https://user-images.githubusercontent.com/9972503/44579937-b6fbfc80-a7ca-11e8-9581-52a373fb24e4.png)

在上述截图中，根据图二的 `token` 变量是能在 `_seenProviders` 中获取到非 `null` 值的，所以会向 `_errors` 中记录一个 `Cannot instantiate cyclic dependency!` 开头的错误。当执行完所有代码之后，控制台会出现该错误：

![interceptor 循环引用报错](https://user-images.githubusercontent.com/9972503/44575832-8ebad080-a7bf-11e8-9578-81a064e4de39.png)

## 用户的修复

那么在 5.2.2 及以前，作为 Angular 开发者，要如何解决上述问题呢？

我们可以通过注入 `Injector` 手动懒加载 `AuthService` 而不是直接注入其到 `constructor`，来使依赖关系变为如下：

```
AuthInterceptor --x-> AuthService -> HttpClient -> AuthInterceptor --x->
即 AuthService -> HttpClient -> AuthInterceptor，其中，在 AuthInterceptor 中懒加载 AuthService
```

即将官方的示例代码修改为如下：

```typescript
import { Injectable, Injector } from '@angular/core';
import { HttpEvent, HttpHandler, HttpInterceptor, HttpRequest } from '@angular/common/http';
import { Observable } from 'rxjs/Observable';
import { AuthService } from '../auth.service';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  private auth: AuthService;

  constructor(private injector: Injector) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    this.auth = this.injector.get(AuthService);

    const authToken = this.auth.getAuthorizationToken();

    const authReq = req.clone({
      headers: req.headers.set('Authorization', authToken)
    });

    return next.handle(authReq);
  }
}
```

可以看到和官方的代码相比，我们改为依赖注入 `Injector`，并通过其实例对象 `this.injector` 在调用 `intercept` 方法时才去获取 `auth` 服务实例，而不是将 `auth` 作为依赖注入、在调用构造函数的时候去获取。

由此我们绕开了编译阶段的对循环依赖做的检查。

## 官方的修复

就像 [PR](https://github.com/angular/angular/pull/19809) 里提到的这样：

> Either HttpClient or the user has to deal specially with the circular dependency.

所以，为了造福大众，最终官方做出了修改，原理和作为用户的我们的代码的思路是一致的——**利用懒加载解决循环依赖问题！**

因为[修复的代码量](https://github.com/angular/angular/commit/ed2b717)很少，所以这里整个摘录下。

首先，新增 `HttpInterceptingHandler` 类（代码一）：

```typescript
// v5.2.3
/**
 * An `HttpHandler` that applies a bunch of `HttpInterceptor`s
 * to a request before passing it to the given `HttpBackend`.
 *
 * The interceptors are loaded lazily from the injector, to allow
 * interceptors to themselves inject classes depending indirectly
 * on `HttpInterceptingHandler` itself.
 */
@Injectable()
export class HttpInterceptingHandler implements HttpHandler {
  private chain: HttpHandler|null = null;

  constructor(private backend: HttpBackend, private injector: Injector) {}

  handle(req: HttpRequest<any>): Observable<HttpEvent<any>> {
    if (this.chain === null) {
      const interceptors = this.injector.get(HTTP_INTERCEPTORS, []);
      this.chain = interceptors.reduceRight(
          (next, interceptor) => new HttpInterceptorHandler(next, interceptor), this.backend);
    }
    return this.chain.handle(req);
  }
}
```

`HttpHandler` 依赖的创建方式由原来的使用 `useFactory: interceptingHandler` 函数（代码二）：

```typescript
// v5.2.2
@NgModule({
  imports: [...],
  providers: [
    HttpClient,
    // HttpHandler is the backend + interceptors and is constructed
    // using the interceptingHandler factory function.
    {
      provide: HttpHandler,
      useFactory: interceptingHandler,
      deps: [HttpBackend, [new Optional(), new Inject(HTTP_INTERCEPTORS)]],
    },
    HttpXhrBackend,
    {provide: HttpBackend, useExisting: HttpXhrBackend},
    ...
  ],
})
export class HttpClientModule {
}
```

改为使用 `useClass: HttpInterceptingHandler` 类（代码三）：

```typescript
// v5.2.3
@NgModule({
  imports: [...],
  providers: [
    HttpClient,
    {provide: HttpHandler, useClass: HttpInterceptingHandler},
    HttpXhrBackend,
    {provide: HttpBackend, useExisting: HttpXhrBackend},
    ...
  ],
})
export class HttpClientModule {
}
```

不难发现，在“代码一”中我们看到了熟悉的写法：依赖注入 `Injector`，并通过其实例对象 `this.injector` 在调用 `handle` 方法时才去获取 `HTTP_INTERCEPTORS` 拦截器依赖，而不是将 `interceptors` 作为依赖注入（在调用构造函数的时候去获取）。

也就是官方修复的思路如下：

```
AuthInterceptor -> AuthService -> HttpClient -x-> AuthInterceptor
即 AuthInterceptor -> AuthService -> HttpClient，其中，在 HttpClient 中懒加载 interceptors
```

因为 `AuthInterceptor` 对 `AuthService` 的引用和 `AuthService` 对 `HttpClient` 的引用是用户定义的，所以官方可以控制的只剩下 `HttpClient` 到拦截器的依赖引用了。所以，官方选择从 `HttpClient` 处切断依赖。

> 那么，我们为什么选择从 `AuthInterceptor` 处而不是从 `AuthService` 处切断依赖呢？
>
> 我觉得原因有二：
>
> 1. 一个是为了让 `AuthService` 尽可能保持透明——对 interceptor 引起的问题没有察觉。**因为本质上这是 interceptors 不能依赖注入 `HttpClient` 的问题。**
> 2. 另一个是 `AuthService` 往往有很多能触发 `HttpClient` 使用的方法，那么在什么时候去通过 `injector` 来 get `HttpClient` 服务实例呢？或者说所有方法都加上相关判断么？……所以为了避免问题的复杂化，选择选项更少（只有一个 `intercept` 方法）的 `AuthInterceptor` 显然更为明智。

## 后记

还是太年轻，以前翻 github 的时候没有及时订阅 issue，导致一些问题修复了都毫无察觉……

从今天起，好好订阅 issue，好好整理笔记，共勉～

> P.S. 好久没写文章了，这篇文章简直在划水……所以我肯定很多地方没讲清楚（特别是代码都没有细讲），各位看官哪里没看明白的请务必指出，我会根据需要慢慢补充。望轻拍砖（逃

## 参考

- [Angular CHANGELOG.md](https://github.com/angular/angular/blob/master/CHANGELOG.md)
- [fix(common): allow HttpInterceptors to inject HttpClient](https://github.com/angular/angular/commit/ed2b717)
- [Insider’s guide into interceptors and HttpClient mechanics in Angular](https://blog.angularindepth.com/insiders-guide-into-interceptors-and-httpclient-mechanics-in-angular-103fbdb397bf)：这篇写得相当得好，深入了拦截器和 `HttpClient` 的内部机制，推荐阅读！