# Django for Android Devs

> Learning backend by debugging a real pagination bug.
>
> Written for someone who knows Android/Kotlin well and has never touched Python or Django. Every backend concept is mapped to something you already use daily.

---

## The setup

A job-board Android app has a bug. The search screen fires a long run of paginated API calls without the user scrolling:

```
GET /api/listings/?offset=0    -> data: []      (empty)
GET /api/listings/?offset=10   -> data: [1 item]
GET /api/listings/?offset=20   -> data: []      (empty)
GET /api/listings/?offset=30   -> data: []      (empty)
...all the way to offset=90
```

Paging3 isn't malfunctioning. It keeps requesting the next page because **the server keeps telling it there is one** — every response carries a non-null `next` field alongside an empty `data` array.

So the client is trusting a lie. To find out why the server is lying, we have to read the server. That's the excuse for this whole document.

*(Names, paths and the domain have been changed throughout. The framework, the architecture, and the bug are real.)*

---

# Lesson 1 — What even *is* a backend?

Your Retrofit interface looks like this:

```kotlin
@GET(BASE_URL + "api/listings/")
suspend fun getListings(...)
```

You've always treated the other end as a black box. The reveal: **the other end is just another program.**

It's a Python program running on a computer in a data centre, sitting in a loop waiting for HTTP requests. When your phone sends one, that program wakes up, runs some functions, builds a JSON blob, and sends it back.

That's a backend. Same as your app — code that takes an input and produces an output. It just runs on a server instead of a phone, and thousands of people hit it at once.

**Django** is the framework for writing that program, the way Jetpack/AndroidX is the framework for writing your app.

## The first question the server must answer

Your phone sends `GET https://api.example.com/api/listings/`.

The server has *hundreds* of endpoints — `/api/listings/`, `/api/listings/detail/`, `/api/listings/health/`, `/api/listings/saved/`. Which code should run?

It needs a lookup table: **URL pattern -> function to call.** You have something similar on Android — a navigation graph, or a deeplink table in the manifest, mapping a URI to a screen.

In Django that table lives in `urls.py`:

```python
from django.urls import re_path
from listings import views

listings_urlpatterns = [
    re_path(r"^health/$", health, name="api_health"),
    ...
    re_path(r"^$", views.RankedListingsView.as_view(), name="api_listings"),
]
```

### Reading that, if you don't know Python

| Python | What it means | Kotlin equivalent |
|---|---|---|
| `from listings import views` | Import | `import listings.views` |
| `listings_urlpatterns = [...]` | A list (square brackets) | `val urlPatterns = listOf(...)` |
| `r"^health/$"` | A raw string holding a **regex** | `"""^health/$""".toRegex()` |
| `re_path(pattern, handler)` | "when the URL matches, call this" | route registration |

So `listings_urlpatterns` is **literally a list of rules**. Django walks it top to bottom, tries each regex against the incoming URL, and stops at the first match.

Now the line our bug hits:

```python
re_path(r"^$", views.RankedListingsView.as_view(), name="api_listings"),
```

That regex `^$` means **"empty string"** — `^` is start, `$` is end, nothing in between. It looks odd, but this whole list is mounted under an `api/listings/` prefix elsewhere. So "empty string *after the prefix*" = exactly `api/listings/` and nothing more.

**That's the endpoint.** `ListingsApi.kt` -> `api/listings/` -> this line -> `RankedListingsView`.

### Check yourself

What does this line do?

```python
re_path(r"^v3/$", views.RankedListingsViewV3.as_view(), name="api_listings_v3"),
```

<details>
<summary>Answer</summary>

URL `api/listings/v3/` runs `RankedListingsViewV3`. It's a *different* class — but as we'll see in Lesson 3, it inherits the same broken response-building code. One bug, two endpoints.
</details>

---

# Lesson 2 — The View

The URL table said: run `RankedListingsView`. Here it is:

```python
class RankedListingsView(generics.ListCreateAPIView):
    authentication_classes = (UserTokenAuthentication,)

    def get(self, request):
        try:
            user_id = request.user_id
            controller = RankedListingsControllerWithSessionId(
                user_id,
                request,
                list_type=ListTypeConfig.MAIN_LIST,
            )
            response_data = controller.get()
            return Response(response_data, status=status.HTTP_200_OK)
        except Exception as e:
            logger.exception(e)
            raise e
```

## Reading Python, line by line

| Python | Kotlin | Note |
|---|---|---|
| `class RankedListingsView(ListCreateAPIView):` | `class RankedListingsView : ListCreateAPIView()` | **Parentheses mean inheritance**, not a constructor call |
| *(indentation)* | `{ }` | Python has **no braces**. Indentation *is* the syntax |
| `def get(self, request):` | `fun get(request)` | `def` = `fun`. `self` = `this`, written out explicitly |
| `try: / except Exception as e:` | `try { } catch (e: Exception) { }` | same idea |
| `raise e` | `throw e` | |

The one that trips up every newcomer: **`self` is an explicit parameter.** Kotlin gives you `this` for free; Python makes you declare it. When you *call* the method you write `view.get(request)` and Python fills in `self` automatically. It's just more honest about what's happening.

## Why is the method called `get`?

This connects straight back to code you've already written.

Your Retrofit interface says `@GET`. That's the **HTTP verb**. Django's base class reads the verb off the incoming request and calls the method with the matching name. `GET` -> `get()`. If the app sent a `POST`, Django would look for a `post()` method on this same class.

So **`@GET` in Kotlin and `def get()` in Python are the two ends of the same wire.** You've been writing one half of this contract for years.

## Where `user_id` comes from

Look at the second line:

```python
authentication_classes = (UserTokenAuthentication,)
```

This is Django's version of an **OkHttp Interceptor.** It runs *before* `get()` does:

```python
def attach_identity(self, request, user_id, app_version):
    setattr(request, "user_id", user_id)
    setattr(request, "app_version", app_version)
```

It reads the auth token, works out who you are, and **staples `user_id` onto the request object.** That's why the view can just say `request.user_id` — someone already put it there.

A few lines up in that same class:

```python
app_version = int(request.META.get("HTTP_X_APP_VERSION"))
```

`request.META` is the HTTP headers. `HTTP_X_APP_VERSION` is a header **the Android app sends on every request.** This is how a backend gates features by app version — all those `if app_version >= 548` checks scattered through the codebase start right here, with a header shipped from the client.

> **Python gotcha:** `(UserTokenAuthentication,)` — that trailing comma isn't a typo. In Python, `(x)` is just `x` in brackets; `(x,)` is a one-item **tuple** (an immutable list). The comma is what makes it a collection.

## The actual point of this lesson

Count the real work in `get()`. Three lines:

```python
controller = RankedListingsControllerWithSessionId(...)  # build it
response_data = controller.get()                         # do everything
return Response(response_data, status=200)               # ship it
```

**The View is deliberately dumb.** It doesn't know what a listing is, doesn't touch a database, doesn't rank anything. It authenticates, delegates, returns.

That's the same discipline you already follow on Android: a Fragment shouldn't hold business logic, it should observe a ViewModel. **`RankedListingsControllerWithSessionId` is the ViewModel here.** Same instinct, different language.

## And the bug is already visible

`response_data` is a plain Python **dict** — key/value pairs, like a Kotlin `Map`. `Response(...)` serializes it straight to JSON.

So this dict:

```python
{"data": [...], "next": "https://...", "count": 137}
```

becomes *exactly* the JSON body you see in Chucker. `data`, `next` and `count` aren't magic framework fields — **they're just keys somebody typed into a dict**, deeper inside the controller.

Which makes the bug findable: something is putting a truthy `next` and an empty `data` into the same dict.

### Check yourself

Another view in the same file uses a different auth class:

```python
class DigestListingsView(generics.ListAPIView):
    authentication_classes = (ServiceTokenAuthentication,)
```

Why?

<details>
<summary>Answer</summary>

Because it isn't called by a user's phone — it's called by **another backend service**. There's no user token, so there's no `user_id` to inject. Different caller, different credential type, different interceptor. Same pattern as using separate OkHttp clients for different APIs.
</details>

---

# Lesson 3 — The Controller

The View delegated to `RankedListingsControllerWithSessionId`. To understand it you need its **parent**, because that's where the shape of the whole request lives.

## First, the constructor

```python
class BaseListingsController(object):
    def __init__(self, request):
        self.user_id = None
        self.request = request
        self._limit_param = None
        self._offset_param = None
        self._listing_ids = []
        ...
```

`__init__` (double underscores, said "dunder init") is **the constructor**. It runs when you write `RankedListingsControllerWithSessionId(...)`.

Here's a real difference from Kotlin. In Kotlin you declare fields and the class has them:

```kotlin
class Foo {
    var listingIds: List<Int> = emptyList()   // declared, exists, typed
}
```

Python has **no field declarations.** An object's fields are whatever you assign to `self.` at runtime. So this constructor, assigning `None` to fifteen things, is doing a job Kotlin's compiler does for you: creating the slots so they exist before anything reads them.

The practical upshot, and it bites people coming from Kotlin: **`self._listing_ids` doesn't exist until some line assigns it.** No compiler will tell you that you forgot. Typo the name and you've silently created a *second* field.

> **Naming convention:** the leading underscore in `_listing_ids` means "private, don't touch from outside." It is *purely* convention. Python will not stop you. There is no `private` keyword.

## The three-step recipe

Now the important part:

```python
def get(self, get_count=False):
    self.process_request()                    # 1. parse the input
    self.fetch_ranked_ids()                   # 2. find WHICH listings
    response = self.build_response_data(...)  # 3. build the JSON
    return response
```

**Every listings endpoint in this service is those three steps.** That's the whole architecture.

This is the **Template Method** pattern — you've used it on Android without naming it. Think `Activity`: the framework defines the sequence (`onCreate` -> `onStart` -> `onResume`) and you override the steps you care about. Same deal:

```
BaseListingsController                 <- defines get() = the 3 steps
  +-- RankedListingsController         <- overrides steps 2 and 3
      +-- ...WithSessionId             <- tweaks step 1
          +-- RankedListingsControllerV3   <- the /v3/ endpoint
```

Nobody down that chain redefines `get()`. They only override individual steps. Which is why **a bug in step 3 hits `/api/listings/` and `/api/listings/v3/` alike** — V3 inherits the broken step without changing it. One bug, multiple endpoints, exactly as the Chucker trace showed.

## Step 1 — `process_request`: unpack the request

```python
(
    self._source_param,
    self._limit_param,
    self._offset_param,
    ...
) = req_handler.extract_params()
```

That's **tuple unpacking** — destructuring. Kotlin has it too:

```kotlin
val (a, b) = pair
```

...but Kotlin caps you at 5 components. Python doesn't care; this one destructures **13 values** in a single positional assignment. Elegant when it's right.

And hold that thought — *"Python binds by position and won't warn you"* comes back to bite hard later.

This step reads `?limit=10&offset=90` off the URL and decodes the filters the app packs into a `data` parameter. Pure parsing. No listings yet.

## Step 2 — `fetch_ranked_ids`: which listings?

This is the expensive, clever part — the search ranking. It hits **Elasticsearch** (the search engine) and **Redis** (a cache), applies relevancy scoring, filters out listings you've already applied to, and produces a ranked list.

And here is **the single most important fact in this whole investigation:**

> ### Step 2 returns only **IDs**. Integers. No content whatsoever.

```python
self._listing_ids = [84213, 91002, 77451, 88190, ...]   # up to ~4,500 of them
```

No titles. No salaries. No company names. Just numbers, in ranked order.

Why? Because ranking 4,500 listings is cheap when they're integers, and ruinously expensive if you drag a full card for each one through the ranking pipeline. So the service ranks IDs, then fetches full content for **only the ~10 about to be shown.**

That's a standard, sensible backend optimisation. You do the same thing when you page a RecyclerView.

## Step 3 — `build_response_data`: build the JSON

Takes the ID list, slices out the 10 for this page, **goes and fetches the actual content for those 10**, and assembles the response dict.

Two separate trips to two separate datastores:

- **Trip 1** (step 2): "which listings?" -> Elasticsearch/Redis -> **IDs**
- **Trip 2** (step 3): "what are these 10?" -> Redis/MongoDB -> **content**

**The bug lives in the gap between those two trips.** Trip 1 says "I found 4,500 listings." Trip 2 says "...I couldn't actually load any of these 10." And nothing reconciles the two answers.

## The alarm that should have caught this

Someone *did* build monitoring for "we returned nothing":

```python
result_count = response.get("count", 0)
if result_count < 1:
    StatsDLogger.incr(metric_name=AppMetric.ZERO_RESULT_COUNT, ...)
```

> **Syntax notes:** `response.get("count", 0)` reads a dict key with a fallback — Kotlin's `map.getOrDefault("count", 0)`. And `f"prefix:{value}"` is an **f-string**, identical to Kotlin's `"$variable"` templates.

This alarm ran through the entire incident and **never fired once.**

### Check yourself

Chucker showed responses with `data: []` — zero listings on screen. So why didn't `ZERO_RESULT_COUNT` fire?

<details>
<summary>Answer</summary>

Because `count` is **not** the number of items in `data`. `count` comes from **trip 1** — it's the length of the ID list, which was a perfectly healthy ~4,500. The empty `data` came from **trip 2**.

The alarm was watching `count` (large, fine) while the actual failure was in `data` (empty). It was structurally incapable of seeing this bug.

**That's why the bug survived in production.** Nobody was measuring the thing that broke.
</details>

---

## The real backend lesson

That last one is worth sitting with, because it's the biggest mindset shift coming from mobile.

On Android, a bug shows up on a screen, in front of a human, and someone screenshots it. On a backend serving millions of requests, **if you didn't add a metric for it, it is invisible** — no matter how badly it's failing. Nobody is looking at your output. There is no screen.

So "add the metric" isn't paperwork you do after the fix. It's how you find out whether the thing was ever broken, and whether your fix worked.

---

# Lesson 4 — Three datastores, and why

On Android you have **one** database. Room. Maybe DataStore for preferences. One place, one source of truth, done.

This service talks to **three**, and the first instinct — "that's over-engineering" — is wrong. They do genuinely different jobs:

| Store | Answers the question | Android-ish analogue |
|---|---|---|
| **MongoDB** | "What *is* listing #84213?" | Room — source of truth |
| **Elasticsearch** | "*Which* listings match 'delivery driver, downtown', ranked?" | *(no real analogue)* |
| **Redis** | "Did we already work this out 30 seconds ago?" | `LruCache`, but shared |

## Why Elasticsearch isn't just a database

Room is brilliant at: *"give me the row where id = 84213."* Indexed, instant.

Room is hopeless at: *"out of 4 million listings, give me the 500 most **relevant** for a user near downtown who wants delivery work, in their language, ranked by how well each one matches, allowing for typos and synonyms — 'delivery boy' should match 'delivery executive'."*

That's not a lookup. That's **scoring every document and sorting by relevance.** A regular database has no concept of "how well does this row match." A search engine does — that's its entire reason to exist.

So: **Elasticsearch decides *which* and in *what order*. MongoDB stores *what each one actually is*.** Two different questions, two different engines. That split is exactly the two trips from Lesson 3.

The Elasticsearch call itself is unremarkable:

```python
self.ranking_query.execute(limit, offset, source=self._source)
```

It comes back with IDs and scores. No content. Just "here are the 500 best matches, in order."

## Why Redis exists — the bit with no mobile equivalent

This is the concept with no Android counterpart, and it's the one that makes everything else click.

Your app runs as **one process on one phone.** A backend runs as **N identical copies** — 50, 100 — behind a load balancer. Your `offset=0` request might hit server #7. Your `offset=10` request, half a second later, hits server #23.

Those are different machines. They share no memory. An in-process `LruCache` would be useless — server #23 has no idea what #7 just computed.

So the cache has to live **outside** all of them: a separate machine every server can talk to. That's **Redis** — a shared in-memory key-value store. Think `Map<String, Any>` that all 50 servers read and write over the network.

That's why caching the ranking matters so much. Elasticsearch scoring 4 million documents is expensive. Redoing it for `offset=10`, `offset=20`, `offset=30` would be insane. So it's computed once, stashed in Redis, and every subsequent page of the scroll reads the *same* cached ranking.

## The read-through cache pattern

You'll recognize this shape immediately — it's the Repository pattern:

```python
def get_listing_content(self):
    _cached_content = RedisCache.get(self.key, machine_alias=self.machine_alias)

    if _cached_content:
        self.listing_content = _cached_content        # cache hit
    else:
        helper = ListingCardUIHelper(self.listing_id, ...)   # miss -> go to MongoDB
        self.listing_content = helper.get_listing_content()
        if self.listing_content:
            self.set_configuration(self.listing_content)     # write it back

    return self.listing_content
```

Check cache -> miss -> fetch from source of truth -> write back -> return. Identical to the Room-plus-network repository you've written a dozen times.

And underneath it:

```python
def get_listing_content_from_mongo(self):
    content = ListingLive().filter(self.language, 1, 0, id=self.listing_id)
    if len(content) > 0:
        content = content[0]

    return content
```

`ListingLive` is the MongoDB collection. **That's the actual source of truth** — the bottom of the stack. Everything above it is caching.

> Note the shape of that function. If the listing is found, `content` becomes the item. If not, it stays the **empty list** it came back as. Park that — it's load-bearing in Lesson 5.

## Cache keys, and the two timers

Both caches declare a key template and a timeout as class constants:

```python
# the ranked ID list (trip 1)
BASE_KEY = "v16:ranked_ids:{user_id}:{list_type}"
TIMEOUT  = 60 * 15

# one listing's content (trip 2)
BASE_KEY = "v6:listing:{listing_id}:{lang}"
TIMEOUT  = 60 * 15
```

Three things to read out of those:

**`{listing_id}:{lang}`** — content is cached *per language*. The same listing in two languages is two separate Redis entries. Nine languages, nine cache entries per listing.

**`v6:` / `v16:`** — a version prefix. When the shape of the cached data changes, you bump the number; every old key is instantly orphaned and ignored. Poor man's cache invalidation. Someone has bumped that ranking one sixteen times.

**`TIMEOUT = 60 * 15`** — **TTL: time to live.** 900 seconds, then Redis deletes the entry. This is the concept with no real Room equivalent, and the one to internalize: **cached data is not permanent, and it is allowed to be stale until it expires.**

## Now — the bug, structurally

You have enough to see it. Two independent caches, each 15 minutes, holding two halves of one answer:

```
Redis:   ranked ID list      [84213, 91002, 77451, ...]   TTL 15 min
Redis:   content for 84213   {title, salary, company}     TTL 15 min
MongoDB: ListingLive         <- live listings only
```

A listing expires. It's removed from `ListingLive` — MongoDB is now correct.

But **the cached ID list still contains its ID**, for up to 15 more minutes. So trip 1 confidently hands over an ID for a listing that no longer exists. Trip 2 goes looking, Redis misses, MongoDB says "not found", and returns that empty list.

And because listings expire **in bulk** at day boundaries, you don't lose one — you lose a contiguous run of them. Sitting at the tail of the relevance ranking, which is exactly where deep offsets land.

 

