# LA15: Deciding Who May

## F1: Two questions

LA14 answered "who are you?": `LoginController` and the `auth` middleware on my five movie routes in `routes/web.php` decide whether somebody is signed in. This week's policy, `app/Policies/MoviePolicy.php`, answers "what may you do?": only an account with the role `admin` may add, edit or delete a movie.

## F2: role and $fillable

`$fillable` only filters mass assignment, where a model is filled from an array like `User::create($validated)`, so a visitor can't slip in a field like `role`. In Tinker I set `$u->role = 'admin'` and called `save()` on one property myself, which is not mass assignment, so `$fillable` didn't apply to it.

## F3: 403 versus login page

A signed-in non-admin typing `/movies/create` gets a 403 from the policy, through `Gate::authorize('create', Movie::class)` in `MovieController`. A guest gets my login page from the `auth` middleware, which runs first. They differ because the guest can fix their problem by logging in, while a non-admin can't, and sending them to login again would be misleading.

## F4: Public list and detail pages

`index` and `show` have no `authorize` call, and `MoviePolicy` returns `true` for `viewAny` and `view`, on purpose. Browsing my movie catalogue is the point of the app, so anyone may look; only changing the data needs permission.

## F5: One rule, two readers

The `$user->role === 'admin'` rule lives only in `MoviePolicy`, and both `MovieController` and `movies/index.blade.php` read it. If I'd written it by hand in both places, changing it (for example adding an editor role) could miss one copy, leaving a button shown that the controller then refuses, or hidden while the controller still allows it.

## F6: SESSION_LIFETIME

`SESSION_LIFETIME` counts the minutes of inactivity since the last request, not the time since I logged in and not the time the page has been open. There was no warning because the server never contacts the browser; the session simply expired, so my next click was treated as a guest request and redirected to login.