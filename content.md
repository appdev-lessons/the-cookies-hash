# The cookies hash

A very special feature of HTTP requests is the ability for the server to store a set of key-value pairs in the client. The storage space in the client is known as "cookies".

In order to store a cookie, the server must include the `Set-Cookie` header in a response:

```http
Set-Cookie: color=purple
```

We can also set multiple cookies at once:

```http
Set-Cookie: color=pink
Set-Cookie: sport=tennis
```

When the client receives a response with headers such as that, it will store the two key-value pairs. Then, in every subsequent request that the client makes, the cookies will be sent back to the server within a `Cookie` header:

```http
GET /dashboard HTTP/1.1
Host: www.example.com
Cookie: color=purple; sport=tennis
```

This opens up some very interesting possibilities. We could, for example, set a cookie within a user's browser that lets us know whether they prefer us to render the page in light mode or dark mode.

### The cookies hash

In Sinatra and Rails, we can set and access cookies using a special hash called `cookies`.

In Sinatra, we must first add `gem "sinatra-contrib"` to our `Gemfile`, `bundle install`, and within `app.rb`:

```ruby
require "sinatra/cookies"
```

Now, within an action, we can use the `cookies` hash with the same old `Hash` methods that we're used to — `.store`, `.keys`, `.fetch`, `[ ]`, etc.

```ruby
get("/zebra") do
  cookies.store("color", "purple")
  cookies.store("sport", "tennis")
  # or, similarly, we can use a more concise technique: []
  cookies["color"] = "purple"
  cookies["sport"] = "tennis"

  # The cookies hash would now look like:
  #    { "color" => "purple", "sport" => "tennis" }
	
  "We stored two values, under the keys 'color' and 'sport'"
end
```

Later, in any view template or within any other action, we can access any cookies that we previously set; even if the user is placing the subsequent request an hour or a day later:

```ruby
get("/girrafe") do
  stored_color = cookies.fetch("color") # returns "purple"
  # or, similarly, we can use a more concise technique: []
  stored_color = cookies["color"] # returns "purple"
	
  "The cookie stored under the key 'color' is #{stored_color}"
end
```

### Signing in is just placing one cookie

Cookies are used for many things — analytics, ad targeting, etc. But the most important thing that we'll use cookies for is _signing in_ a user.

When a fills out a sign in form, on the backend we'll check in our database to make sure the password matches the email address they provided. If so, we'll store their user ID in a cookie.

Then, with every subsequent request from that browser, we'll know who they are; and we can customize the response accordingly. So, all being "signed in" means is having that one cookie set, which is why clearing your browser cookies signs you out of every site that you were signed in to.

We'll get to that in a later lesson, after we learn about databases.

### Storing complex values

The keys and values we can store within cookies can only be strings. HTTP does not have official support for any other data types — no `Time`, `Array`, `Hash`, etc. So, if we want to store those more complicated types of objects, we must first _encode_ them as a string. Later, after we pull the encoded strings out of `cookies`, we must transform them back into the original objects.

Most commonly, we will want to store an `Array` or `Hash` of values within a single cookie. For that, a useful trick is to convert the Ruby `Array` or `Hash` into a JSON string, and then store the string.

This is the inverse of what we did in the past when working with APIs: back then we'd receive a JSON string, and we'd want to convert into Ruby objects. Back then, we used `JSON.parse()`, which accepts a `String` containing valid JSON as an argument and returns an `Array` or `Hash`.

```ruby
require "json"

original_string_containing_json = '{ "sport": "tennis", "color": "purple" }'

parsed = JSON.parse(original_string_containing_json)

pp original_string_containing_json.class

pp original_string_containing_json

pp parsed.class

pp parsed
```
{: .repl #json_parse title="JSON parse" points="1"}

To go in the other direction, we can use the inverse method, `JSON.generate()`, which accepts an `Array` or a `Hash` as an argument and returns a `String`:

```ruby
require "json"

original_hash = { "fruit" => "mango", "city" => "chicago" }

flattened = JSON.generate(original_hash)

pp original_hash.class

pp original_hash

pp flattened.class

pp flattened
```
{: .repl #json_generate title="JSON generate" points="1"}

This technique is very useful; for example, we could store an array of the past 5 calculations that a user has done and display a history for them.
