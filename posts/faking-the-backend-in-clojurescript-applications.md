# Faking the backend in clojurescript applications

Whenever I have to do any frontend work, I always like to decouple it from the backend.
It encourages a clean contract specification, allows running and developing
the UI without a backend, and enables many UI exploration options.  

What happens if network is slow? If text is too short or too long? If data is corrupted?
All these scenarios are much easier to explore (and test) if the backend is faked.

My tool of choice is [msw](https://mswjs.io). Implemented on service workers, it allows
me to fake the backend without any code change or without having to spin up another application.

```clojure
(ns dev.fake.browser
  (:require [app.utils.localstorage :as lc]
            [dev.fake.foo :as fake.foo]
            ["msw" :refer (setupworker)]))

(def ^:private mock-key "mock-active?")
(def ^:private ^js/object worker (apply setupworker fake.foo/fakes))

(defn- start-worker! []
  (.start worker (clj->js {:onunhandledrequest "bypass"})))

(defn fake-start! []
  (->  (start-worker!)
       (.then #(lc/set-item! mock-key true))))

(defn fake-stop! []
  (.stop worker)
  (lc/remove-item! mock-key))

(defn fake-init! []
  (if (lc/get-item mock-key)
    (start-worker!)
    (js/promise.resolve)))
```

You can now start/stop your fake service from the repl. Also, don't forget to 
`(fake-init!)` when starting the application.That's going to keep the fakes
between page reloads.

(more to come)
