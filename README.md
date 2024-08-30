# content-fetch

A method for web extensions to execute fetch requests from content scripts in background scripts.

js

```js
function retry(fn, attempts = 3, delay = 3000) {
  return async (...args) => {
    for (let i = 0; i < attempts; i++) {
      try {
        return await fn(...args);
      } catch (e) {
        if (i < attempts - 1) {
          await new Promise(r => setTimeout(r, delay));
        } else {
          throw e;
        }
      }
    }
  };
}

export function initContentFetchInContentScript() {
  browser.runtime.onMessage.addListener(async ({ type, input, init }) => {
    if (type === 'CONTENT_FETCH') {
      try {
        const response = await fetch(input, init);
        return {
          body: await response.text(),
          status: response.status,
          statusText: response.statusText,
          headers: Object.fromEntries(response.headers.entries()),
        };
      } catch (error) {
        return { error: error.message };
      }
    }
  });
}

export function initContentFetchIn(origin) {
  async function getTab() {
    const tabs = await browser.tabs.query({ url: `${origin}/*` });
    if (tabs.length > 0 && tabs[0].id) {
      return tabs[0].id;
    }
    const tab = await browser.tabs.create({ url: origin, active: false });
    return tab.id;
  }

  async function contentFetch(input, init) {
    const tabId = await getTab();
    const message = { type: 'CONTENT_FETCH', input: input.toString(), init };
    const sendMessageWithRetry = retry(async () => {
      const response = await browser.tabs.sendMessage(tabId, message);
      return new Response(response.body, {
        status: response.status,
        statusText: response.statusText,
        headers: new Headers(response.headers),
      });
    });
    return sendMessageWithRetry();
  }

  return contentFetch;
}
```

exmaple

```js
// content script
initContentFetchInContentScript()

// background script
const exampleContentFetch = initContentFetchIn('https://example.com')
exampleContentFetch('https://example.com/api', {
  headers,
  method: 'GET',
}
```

ts

```ts
function retry(fn: any, attempts = 3, delay = 3000) {
  return async (...args: any) => {
    for (let i = 0; i < attempts; i++) {
      try {
        return await fn(...args)
      }
      catch (e) {
        if (i < attempts - 1)
          await new Promise(r => setTimeout(r, delay))
        else throw e
      }
    }
  }
}

export function initContentFetchInContentScript() {
  browser.runtime.onMessage.addListener(async ({ type, input, init }) => {
    if (type === 'CONTENT_FETCH') {
      try {
        const response = await fetch(input, init)
        return {
          body: await response.text(),
          status: response.status,
          statusText: response.statusText,
          headers: Object.fromEntries(response.headers.entries()),
        }
      }
      catch (error: any) {
        return { error: error.message }
      }
    }
  })
}

export function initContentFetchIn(origin: string) {
  async function getTab() {
    const tabs = await browser.tabs.query({ url: `${origin}/*` })
    if (tabs.length > 0 && tabs[0].id) {
      return tabs[0].id
    }
    const tab = await browser.tabs.create({ url: origin, active: false })
    return tab.id as number
  }

  async function contentFetch(input: RequestInfo, init?: RequestInit) {
    const tabId = await getTab()
    const message = { type: 'CONTENT_FETCH', input: input.toString(), init }
    const sendMessageWithRetry = retry(async () => {
      const response = await browser.tabs.sendMessage(tabId, message)
      return new Response(response.body, {
        status: response.status,
        statusText: response.statusText,
        headers: new Headers(response.headers),
      })
    })
    return sendMessageWithRetry()
  }

  return contentFetch
}
```
