**While this is an alpha release, the code is well-tested and pretty stable. We encourage you to test it out and [report any issues](https://github.com/sindresorhus/got/issues/876). However, we don't recommend TypeScript users to use this yet as [the types are incomplete](https://github.com/sindresorhus/got/issues/758).**

### Breaking

- Remove support for protocol-less URLs in the `url` argument  92bc808
	- Why: To reduce ambiguity. It was not clear from just reading the code what it would default to.
	- Migrate:
```diff
- got('sindresorhus.com');
+ got('https://sindresorhus.com');
```
- Rename the `query` option to `searchParams` and make it stricter  b223663 5376216
	- Note: The `query` option name is still supported, but it will be removed in the next major version.
	- Why: To get closer to the `window.fetch` naming in the browser.
	- Migrate:
```diff
- got(…, {query: …});
+ got(…, {searchParams: …});
```
- Replace the `baseUrl` option with `prefixUrl` (#829)  0d534ed
	- Note: We also made it stricter to reduce ambiguity. The Got `url` argument now cannot be prefixed with a slash when this option is used.
	- Why: To make it clear that it doesn't do any URL resolution.
	- Migrate:
```diff
- got('/foo', {baseUrl: 'https://x.com'});
+ got('foo', {prefixUrl: 'https://x.com'});
```
- Change the `json` option to accept an object instead of a boolean and to only be responsible for the request, not the response (#704)  a6a7d5a
	- Note: You now set the request body in this option instead of the `body` option when you want to send JSON. This option also no longer sets the response type to JSON. You either call the `.json()` method or specify the `responseType` option for that.
	- Why: Many people were confused how `{json: true}` worked and they also complained that they could not set the request/response individually.
	- Migrate:
```diff
- got(url, {body: {x: true}, json: true});
+ got.post(url, {json: {x: true}}).json();
```
- Don't infer `POST` automatically when specifying `body` (#756)  e367bdb
	- Why: We're trying to reduce the amount of magic behavior.
	- Migrate:
```diff
- got(…, {body: 'foo'});
+ got.post(…, {body: 'foo'});
```
- The `retries.retry` option was split into `retries.limit` and `retries.calculateDelay` b15ce1d
	- Migrate:
```diff
 got(…, {
 	retry: {
-		retries: 2
+		limit: 2
 	}
 });
```diff
 got(…, {
 	retry: {
-		retries: iteration => iteration < 2
+		calculateDelay: ({attemptCount}) => attemptCount < 2
 	}
 });
```
- Rename the Promise API property `.fromCache` to `.isFromCache` (#768)  b5e443b
- Move top-level error properties into an `.options` and `.response` property (#773)  6eaa81b
	- Migrate:
```diff
- error.gotOptions
+ error.options

- error.headers
+ error.response.headers

- error.statusCode
+ error.response.statusCode

- error.statusMessage
+ error.response.statusMessage

- error.body
+ error.response.body

- error.redirectUrls
+ error.response.redirectUrls

- error.host
+ error.options.host

- error.hostname
+ error.options.hostname

- error.method
+ error.options.method

- error.protocol
+ error.options.protocol

- error.url
+ error.options.url

- error.path
+ error.options.path
```
- Custom instance creation was simplified (#707)  8eaef94
	- Note: `got.mergeInstances(...instances)` is deprecated. Use `instanceA.extend(instanceB)` instead.
	- Migrate:
```diff
- got.create({handler: handler});
+ got.create({handlers: [handler]});

# Merging instances
- instanceA.extend(instanceB, instanceC, …);
+ got.mergeInstances(instanceA, instanceB, instanceC, …);

# Merging options
- instanceA.extend(optionsB, optionsC, …)
+ instanceA.extend(optionsB).extend(optionsC).extend(…);

# Merging instances and options
- instanceA.extend(optionsB, instanceC, …);
+ got.mergeInstances(instanceA.extend(optionsB), instanceC);

# Extending handlers
- instanceA.extend({handlers: [handlerB]});
+ got.mergeInstances(instanceA, got.create({handler: handlerB}));
```

*Note: The notes here will be expanded in the final release.*

### Enhancements

- Got has been written in TypeScript.
	This means we can provide our own type definitions and we can be more confident when working on the Got codebase and produce less bugs.
- Add support for [Brotli](https://en.wikipedia.org/wiki/Brotli) (Node.js 12 and later) (#706)  d5d2e6f
- Add opt-in [DNS cache](https://github.com/sindresorhus/got#dnscache) (#731)  cd12351
- Add [`context` option](https://github.com/sindresorhus/got#context) for storing custom metadata across request and hooks (#777)  3bb5aa7
- Add [option to ignore invalid cookies](https://github.com/sindresorhus/got#ignoreinvalidcookies) (#826)  e9c01e0
- Proxy headers from server request to Got (#772)  00e5fd5
- Pass the response as the second argument to the `beforeRedirect` hook (#812)  3557896
- Make `URLSearchParams` instances mergeable (#734)  95c7c2c
- Throw on canceled request with incomplete response (#767)  92b1005
- Add `[.isFromCache` property](https://github.com/sindresorhus/got#streams-1) to the stream API (#768)  b5e443b

### Fixes

- Fix parsing response when using `afterResponse` hook (#775)  e2054cd
- Fix `port` not being reset on redirect (#729)  ada5861
- Fix the retry functionality (#787)  0501e00
- Fix default `retry` option value when specifying a number (#809)  9c04a7c
- Correctly handle promise- and stream-specific errors in the `beforeError` hook  134c9b7
- Don't throw on early lookups  4faf5c7

### Docs

- Document that `retry` option doesn't work with streams  9088866
- Encourage using `Stream.pipeline()` when using the stream API  06afb27
- Add instructions for `global-agent` (#822)  ca8c560
- Mention the `TimeoutError.timings` property  8fa18f4
