---
layout: post
title: Create a Rails 4 site with a simple contact us form
tagline: a complete tutorial
category: articles
---

<section id="table-of-contents" class="toc">
  <header>
    <h3 class="delta">Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

This tutorial uses [Twitter Bootstrap][bootstrap], [bootstrap-sass][bootstrap-sass], [SimpleForm][simple_form], [MailForm][mail_form], and [Guard][guard].
If you are familiar with Rails basics and [bootstrap-sass][bootstrap-sass], you can skip to the <a href="#add-contact-us-form">Add contact us form</a> section

## 1. Create a Rails 4 Project
{% highlight bash %}
$ rails new project_name
$ cd project_name
{% endhighlight %}

Edit `Gemfile`. Add the following gems.

{% highlight ruby %}
# Front-end {
gem 'bootstrap-sass', '~> 2.3.2.0'
gem 'haml-rails', '~> 0.4.0'
# }
# Forms, mail {
gem 'mail_form', '~> 1.5.0.rc'
gem 'simple_form', '~> 3.0.0.rc'
# }
# Development (Optional) {
gem 'better_errors', group: :development
gem 'quiet_assets', group: :development
# }
# Development Guard {
gem 'guard-rails', group: :development
gem 'guard-livereload', group: :development
gem 'rack-livereload', group: :development
gem 'guard-bundler', group: :development
# }
{% endhighlight %}

See the [example Gemfile with extra gems such as pg, bootswatch, heroku, font awesome](https://gist.github.com/l4u/5945453)

Install gems by running
{% highlight bash %}
$ bundle install
{% endhighlight %}

### Setup guard

We use guard to automate gem installations, livereload, rails server restarts, etc.

Add guard definition to your Guardfile by running
{% highlight bash %}
$ guard init rails bundler livereload
{% endhighlight %}

You should see

{% highlight text %}
08:21:14 - INFO - rails guard added to Guardfile, feel free to edit it
08:21:14 - INFO - bundler guard added to Guardfile, feel free to edit it
08:21:14 - INFO - livereload guard added to Guardfile, feel free to edit it
{% endhighlight %}

### Booting Rails server with guard

Start Rails server and livereload by running
{% highlight bash %}
guard
{% endhighlight %}

Visit http://localhost:3000 in your browser. You should see the default Rails welcome page.

![Rails 4 welcome page](/images/2013-07-08-rails-welcome-page.png)

## 2. Create a home controller
{% highlight bash %}
rails g controller home index
{% endhighlight %}

![rails generate controller results](/images/2013-07-08-rails-generate.png)


Edit `config/routes.rb`

Replace `get "home/index"` with `root "home#index"`
So that when you visit http://localhost:3000, the request will be handled by the home controller.

After saving the file, guard should restart your Rails server.
Check if there's any errors in the guard console.
Visit http://localhost:3000 and you should see

![Home#index Find me in app/views/home/index.html.haml](/images/2013-07-08-rails-home-controller.png)

## 3. Add styles

### Add Bootstrap JavaScript and SCSS

We need some styles! Let's add Twitter bootstrap.

Add in `config/application.rb`
{% highlight ruby %}
config.assets.precompile += %w(*.png *.jpg *.jpeg *.gif)
{% endhighlight %}

Rename  `app/assets/stylesheets/application.css`
to `app/assets/stylesheets/application.css.scss`

Add in `app/assets/stylesheets/application.css.scss`
{% highlight scss %}
@import "bootstrap";
{% endhighlight %}

Add in `app/assets/javascript/application.js`, before `//= require_tree .`
{% highlight ruby %}
//= require bootstrap
{% endhighlight %}



The page should now look like

![home controller with bootstrap style](/images/2013-07-08-rails-home-controller-bootstrap.png)

### Convert erb to Haml
The default Rails uses erb. Now we convert it to haml by [html2haml](https://github.com/haml/html2haml)

You can install html2haml as:
{% highlight bash %}
$ gem install html2haml
{% endhighlight %}

Convert the default layout in erb to haml
{% highlight bash %}
cd app/views/layouts
html2haml application.html.erb > application.html.haml
{% endhighlight %}

Remove the erb file
{% highlight bash %}
rm application.html.erb
{% endhighlight %}

### Add Bootstrap HTML and navigation bar

Edit `app/views/layout/application.html.haml`
Add between `%body` and `=yield`
{% highlight haml %}
.navbar.navbar-fixed-top
  .navbar-inner
    .container
      %button.btn.btn-navbar{"data-target" => ".nav-collapse", "data-toggle" => "collapse", :type => "button"}
        %span.icon-bar
        %span.icon-bar
        %span.icon-bar
      %a.brand{:href => root_url}
        brand
      .nav-collapse.collapse
        %ul.nav.pull-right
          %li
            = link_to t('Contact'), "todo"
{% endhighlight %}

Because we are using fixed navbar in this example, append `app/assets/stylesheets/application.css.scss`
{% highlight haml %}
body {
  padding-top: $navbarHeight + 20px;
}
{% endhighlight %}

Edit `app/views/home/index.html.haml`
{% highlight haml %}
.container
  .row
    .span12
      %h1 Home#index
      %p Find me in app/views/home/index.html.haml
{% endhighlight %}

Your page should look like this now.

![Style home controller with a navigation bar](/images/2013-07-08-rails-home-controller-styled.png)

## 4. Add contact us form

Run the simple_form generator

{% highlight bash %}
rails g simple_form:install --bootstrap
{% endhighlight %}

### Controller

{% highlight bash %}
rails g controller contacts
{% endhighlight %}

Edit `config/routes.rb`, remove the generated routes for `contacts`. Add
{% highlight ruby %}
resources "contacts", only: [:new, :create]
{% endhighlight %}

Replace `app/controllers/contacts_controller.rb` with
{% highlight ruby %}
class ContactsController < ApplicationController
  def new
    @contact = Contact.new
  end

  def create
    @contact = Contact.new(params[:contact])
    @contact.request = request
    if @contact.deliver
      flash.now[:error] = nil
      flash.now[:notice] = 'Thank you for your message!'
    else
      flash.now[:error] = 'Cannot send message.'
      render :new
    end
  end
end
{% endhighlight %}

### Model

Create `app/models/contact.rb` with
{% highlight ruby %}
class Contact < MailForm::Base
  attribute :name,      :validate => true
  attribute :email,     :validate => /\A([\w\.%\+\-]+)@([\w\-]+\.)+([\w]{2,})\z/i
  attribute :message
  attribute :nickname,  :captcha  => true

  # Declare the e-mail headers. It accepts anything the mail method
  # in ActionMailer accepts.
  def headers
    {
      :subject => "My Contact Form",
      :to => "your_email@example.org",
      :from => %("#{name}" <#{email}>)
    }
  end
end
{% endhighlight %}
(Trimmed version of the [MailForm example](https://github.com/plataformatec/mail_form))

### Views

Create `app/views/contacts/new.html.haml` with
{% highlight ruby %}
.container
  %h1 Contact
  = simple_form_for @contact, :html => {:class => 'form-horizontal' } do |f|
    = f.input :name, :required => true
    = f.input :email, :required => true
    = f.input :message, :as => :text, :required => false, :input_html => {:rows => 10}

    .hidden
      = f.input :nickname, :hint => 'Leave this field blank!'
    .form-actions
      = f.button :submit, 'Send message', :class=> "btn btn-primary"

{% endhighlight %}
Create `app/views/contacts/create.html.haml` with
{% highlight ruby %}
.container
  %h1 Thank you for your message.
  %p We'll get back to you soon.
{% endhighlight %}

Add in `app/assets/stylesheets/application.css.scss`
{% highlight scss %}
.hidden { display: none; }
{% endhighlight %}

Edit `app/views/layout/application.html.haml`, add/edit the menu item
{% highlight haml %}
%li
  = link_to t('Contact'), new_contact_path
{% endhighlight %}

Add flashes just after the `%body`
{% highlight haml %}
.container
  - flash.each do |name, msg|
    - if msg.is_a?(String)
      %div{:class => "alert alert-#{name == :notice ? "success" : "error"}"}
        %a.close{"data-dismiss" => "alert"}
          &times;
        = content_tag :div, msg, :id => "flash_#{name}"
{% endhighlight %}

### Results

Open [http://localhost:3000/contacts/new](http://localhost:3000/contacts/new) in your browser

You should see the working contact us form with server-side validations.

![Rails 4 Mail form with validations](/images/2013-07-08-rails-mail-form-validation.png)

For real e-mail delivery, you should set up SMTP in the environment settings.

# Bonus / Further readings
- Add tests for mail delivery
- Add [sidekiq][sidekiq], [resque][resque], or [delayed_job][delayed_job] for background mail delivery
- Add [bootswatch-rails][bootswatch-rails] for easy theming.
- Use [Rails Apps Composer](https://github.com/RailsApps/rails_apps_composer) for automations

[bootstrap]: http://twitter.github.io/bootstrap/
[bootstrap-sass]: https://github.com/thomas-mcdonald/bootstrap-sass
[simple_form]: https://github.com/plataformatec/simple_form
[mail_form]: https://github.com/plataformatec/mail_form
[guard]: https://github.com/guard/guard
[bootswatch-rails]: https://github.com/maxim/bootswatch-rails
[sidekiq]: http://sidekiq.org
[resque]: https://github.com/resque/resque
[delayed_job]: https://github.com/collectiveidea/delayed_job
