# Data fetching with React Query

[React Query](https://tanstack.com/query/v3/) is a library that takes away the burden of managing remote state. It also has a nice hierarchical cache invalidation mechanism and optimistic updates with rollbacks.

If all your app is doing is fetching data from a database and displaying it on a screen, then React Query is the right choice here.

```clojure
(ns my.app
  (:require [uix.core :as uix]
            [cljs-bean.core :as bean]
            [clojure.string :as str]
            [react-query :as rc]))

;; create the client
;; it's a shared global state that maintains cache
(defonce query-client (rc/QueryClient.))

(defn camel-case->kebab-case [s]
  (-> s
      (str/replace #"(?<=[a-z])(?=[A-Z])" "-")
      (str/lower-case)))

;; a wrapper for useQuery hook
(defn use-query [{:keys [key f]}]
  (bean/->clj
    (rc/useQuery #js {:queryKey (clj->js key) :queryFn f})
    :prop->key camel-case->kebab-case))

(defn fetch [url]
  (-> (js/fetch url)
      (.then #(.json %))
      (.then #(js->clj % :keywordize-keys true))))

(defn fetch-articles
  (fetch "..."))

(defui articles []
  (let [{:keys [data is-loading is-error is-success]}
        (use-query {:key [:articles]
                    :f fetch-articles})]
    (cond
      is-loading ($ :div "Loading...")
      is-error ($ :div "Couldn't load")
      is-success ($ :div "{display articles here}"))))

(defui article-by-id [{:keys [id]}]
  (let [{:keys [data]} (use-query {:key [:articles id]
                                   :f #(fetch-article-by-id id)})]
    "{display the article here}")

(defui root []
  ($ rc/QueryClientProvider {:client query-client}
    ($ articles)
    ($ article-by-id {:id 1})))
```
