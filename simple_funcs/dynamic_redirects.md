# How to redirect_to the last location

In your edit action, stroe the requesting url in the session_hash, which is available across multiple requests:
```ruby
session[:return_to] ||= request.referer
```

Then redirect to it in your update action, after a successful save:
```ruby
redirect_to seesion.delete(:return_to)
```
