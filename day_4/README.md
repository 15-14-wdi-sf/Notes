# Facebook Authentication and Open Graph
## Facebook Authentication
[This Railscast](http://railscasts.com/episodes/360-facebook-authentication) has everything you need.

Two gotchas:

- Instead of pointing the Facebook login route to sessions#create, you might want to point that at a new action in the sessions controller or a new controller altogether
- Modify the migration Ryan runs in this Railscast to look like this:

```shell
rails g migration add_oauth_fields_to_users provider uid name oauth_token oauth_expires_at:datetime
```

- Modify Ryan's `from_omniauth` method in the User model to this (it will make sure your user can be saved even without a password):

```ruby
def self.from_omniauth(auth)
  where(uid: auth.uid).first_or_initialize.tap do |user|
    user.provider = auth.provider
    user.uid = auth.uid
    user.name = auth.info.name
    user.oauth_token = auth.credentials.token
    user.oauth_expires_at = Time.at(auth.credentials.expires_at)
    user.save!(validate: false) # skip password validation
  end
end
```

## Facebook Graph
Let's say we want to interact with the user's graph and post a new TODO on his timeline whenever he posts one in our app. We're going to use [Koala](https://github.com/arsduo/koala) for that.
