MailView -- Visual email testing
================================

This is a fork from 37signals MailView gem (now basecamp gem i think), i'll add some changes to make the interface more flexible with the posibility to add groups

Now you can use the mailer in different locales by tracking the Top Level Domain (TLD).

TODO: sort & search functions

Preview plain text and html mail templates in your browser without redelivering it every time you make a change.

Install
-------

Add the gem to your `Gemfile`:

```ruby
  gem 'mail_view', :git => 'https://github.com/LucasPadovan/mail_view.git'
```

And run `bundle install`.

Then you should create the following file.
Title and Extended Title are used in the index of the mail view.
Setting true the USE_TLD constant will allow you to track the top level domain for localization purposes

```ruby
# config/initializers/mail_preview.rb
class MailView
  TITLE                = 'Title'
  EXTENDED_TITLE       = 'Extended title'
  USE_TLD              = true
  MAIL_PREVIEW_COUNTRY = ''
end
```

Usage
-----

Since most emails do something interesting with database data, you'll need to write some scenarios to load messages with fake data. Its similar to writing mailer unit tests but you see a visual representation of the output instead.

```ruby
  # app/mailers/mail_preview.rb or lib/mail_preview.rb
  class MailPreview < MailView
    # Pull data from existing fixtures
    def groupName_invitation
      account = Account.first
      inviter, invitee = account.users[0, 2]
      Notifier.invitation(inviter, invitee) 
    end

    # Factory-like pattern
    def groupName_welcome
      user = User.create!
      mail = Notifier.welcome(user)
      user.destroy
      mail
    end

    # Stub-like
    def groupName_forgot_password
      user = Struct.new(:email, :name).new('name@example.com', 'Jill Smith')
      mail = UserMailer.forgot_password(user)
    end
  end
```

Methods must return a [Mail][1] or [TMail][2] object. Using ActionMailer, call `Notifier.create_action_name(args)` to return a compatible TMail object. Now on ActionMailer 3.x, `Notifier.action_name(args)` will return a Mail object.

Routing
-------

A mini router middleware is bundled for Rails 2.x support.

```ruby
  # config/environments/development.rb
  config.middleware.use MailView::Mapper, [MailPreview]
```

For Rails³ you can map the app inline in your routes config.

```ruby
  # config/routes.rb
  if Rails.env.development?
    mount MailPreview => 'mail_view'
  end
```

Now just load up `http://localhost:3000/mail_view`.

Interface
---------

![Plain text view](http://img18.imageshack.us/img18/1066/plaintext.png)
![HTML view](http://img269.imageshack.us/img269/2944/htmlz.png)

Using localization with TLD
---------------------------

In your initializer you should have
```ruby
USE_TLD = true
```

then you can do something like this in your scenarios

```ruby
# app/mailers/mail_preview.rb or lib/mail_preview.rb
    #[...]

  def stub_mailer
    user = User.where(where_country).last
    UserMailer.deliver(user)
  end

  private
    def country
      I18n.locale = :es
      case MAIL_PREVIEW_COUNTRY
        when 'br'
          I18n.locale = :pt
          'Brasil'
        when 'cl'
          'Chile'
        when 'co'
          'Colombia'
        when 'mx'
          'Mexico'
        else
          'Argentina'
      end
    end

    def where_country
      ['country = ?', country]
    end
```

and this way you can test your localized templates that depends on the country of the User.

[1]: http://github.com/mikel/mail
[2]: http://github.com/mikel/tmail


Disclaimer: the TLD functionality was created for a very specific architecture,
maybe you have to tweak a little your mailers to get full functionality.
Feel free to contact me or open issue tickets.

Regards
LucasPadovan
