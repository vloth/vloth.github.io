# Live change of a fake state

In the [last post](/#/post/faking-the-backend-in-clojurescript-applications) we saw how easy it is
to enable/disable the fake setup of your client application from repl in clojurescript applications.  
But how could we achieve a similar experience when working in js/ts based applications?  

### The journey

The [documentation suggests](https://mswjs.io/docs/getting-started/integrate/browser#start-worker) starting the worker
when NODE_ENV is development. While that works well for an initial setup, it takes away the chance to run the application
in development mode with a real backend service.

So we decide to toggle based on a new variable, something like REACT_APP_MSW, and supply two different scripts:
```ts
// package.json
{
  "scripts": {
    "start": "...",
    "start:msw: "REACT_APP_MSW=true npm start"
  }
}

```
npm start can be used to run the client with a real backend server, while npm run start:msw can be used to run
with a faked backend.

While this setup might work well, it's not ideal as we would have to kill the process and restart the client every time
we want to change between backend implementations.

### A possible solution

We could just update the global window variable with the worker instance and connect it to a local storage. 
The developer will be able to controle it from the browser's console.
This was suggested before [here](https://github.com/mswjs/msw/issues/123).

But the browser console isn't really part of a js/ts developer workflow (beyond inspecting the page).
A possible solution is just connecting the worker instance to a developer page available only
when in dev mode. This would allow the developer to toggle the fake state, and it creates a solid foundation
for developer guided tooling. Useful links, tips, and more can be done from there.

```ts
//mocks/browser.ts
import { setupWorker } from 'msw'

import { handlers } from './handlers'

const worker = setupWorker(...handlers)

export const init = () => {
    if (localStorage.getItem('mock-enabled?')) worker.start()
}

export const start = () => {
    localStorage.setItem('mock-enabled?', String(true))
    worker.start({ quiet: true })
    return true
}

export const stop = () => {
    localStorage.removeItem('mock-enabled?')
    worker.stop()
    return false
}

export const isEnabled = () =>
    localStorage.getItem('mock-enabled?') === String(true)

export const toggle = () => (isEnabled() ? stop() : start())
```

```tsx
// pages/Development.tsx
const [mockEnabled, setMockValue] = useState(MswBrowser.isEnabled())

return (
  <label>
    <input type="checkbox" checked={mockEnabled} onChange={() => setMockValue(MswBrowser.toggle())} />
    Enable mocks?
    </Typography>
  </label>
)
```

This should give a good developer experince, since the fake state can be toggled without restarting the app:
The above can also be done for clojurescript applications, of course. Although developers (including myself) would
probably just prefer using the repl.
