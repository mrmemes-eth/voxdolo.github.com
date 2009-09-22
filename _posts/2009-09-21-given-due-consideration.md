---
title: Given Due Consideration
layout: post

---

## Or: Just Say No to Cukes

As alluded to by [veez](http://blog.veez.us/2009/09/11/integration-testing-without-cucumber),
the undercurrent from some of us at [Hashrocket](http://www.hashrocket.com) has
been decidedly anti-[cucumber](http://cukes.info). Here's my brief littany:

* A well developed cucumber suite is invariably slow to run
* For me, at least, it makes development feel slower (this is almost certainly
  perceptual, but that doesn't mean it's not actual as well)
* The indirection makes debugging laborious
* Matching plain english to executable code via regular expressions just 
  *feels wrong*

All of these things (and more) have been excused for quite some time because
Cucumber is thought of as a necessary tax. If you want to do full stack
integration testing in Ruby... this is how it's done.

Here's the good news: it's not the only way! Veez has gone over the basics in
the post I linked above, so I won't belabor the setup, though I will give you
the latest version. Here are the relevant bits of my `spec/spec_helper.rb`:

{% highlight ruby %}
  require 'webrat'
  require 'spec/support/integration'

  Webrat.configure do |config|
    config.mode = :rails
  end

  Spec::Runner.configure do |config|
    config.include(Webrat::Matchers, :type => [:integration])
  end

  class ActionController::Integration::Session; include Spec::Matchers; end
{% endhighlight %}

Interleaving the above into your existing `spec_helper.rb` is all you need to
get going.

Let's use this user story from a project I'm working on now as the backdrop to
our discussion:

> In order to dissolve a business relationship  
> As a user  
> I want to be able to remove a colleague
> 
> Scenario:  
> Given a user with colleagues  
> When I click a colleagues "Remove colleague" button  
> Then I should be on the colleagues list  
> And I should not see the colleague  
> And the colleague should not see me in their colleague list

Here's how you might implement that using RSpec and Webrat:

{% highlight ruby %}
  describe "User removes colleague" do
    before do
      login_as @user
      visit colleauges_path
      click_button 'Remove colleague'
    end

    it "redirects you to the colleagues list" do
      current_url.should == colleagues_url
    end

    it "removes them from my colleagues list" do
      response.body.should_not have_tag('#colleagues .colleague')
    end

    it "removes me from their colleague list" do
      logout
      login_as(@colleague)
      visit colleauges_path
      response.body.should_not have_tag('#colleagues .colleague')
    end
  end
{% endhighlight %}

That's all well and good, but you may be saying to yourself (or your co-worker,
or your poor, terrified cat): "But, that doesn't capture the plain english user
story! Darn it all to Bethesda!". And you're absolutely right... but how's this
strike you:

{% highlight ruby %}
  Feature "User removes colleague" do
    Given do
      login_as @user
      visit colleagues_path
      click_button 'Remove colleague'
    end

    Scenario "removal" do
      When "I press remove" do
        Then "I should be on my collegues list" do
          current_url.should == colleagues_url
        end

        And "I should not see the user in my colleagues list" do
          response.body.should_not have_tag('#colleagues .colleague')
        end

        And "My colleague should not see me in their colleague list" do
          logout
          login_as @colleague
          visit colleagues_path
          response.body.should_not have_tag('#colleagues .colleague')
        end
      end
    end
  end
{% endhighlight %}

[@l4rk](http://twitter.com/l4rk) and I came up with this clever bit of aliasing
to get you where you want to go. Just add this at the bottom of your
`spec_helper.rb`:

{% highlight ruby %}
  module Spec::DSL::Main
    alias :Feature :describe
  end

  module Spec::Example::ExampleGroupMethods
    alias :Scenario :describe
    alias :When :describe
    alias :Given :before #TODO: make Given take :before or :after and :each or :all
    alias :Then :example
    alias :And :example
  end
{% endhighlight %}

The failing of this technique, for the time being, is the round-trip output:

> User removes colleague removal I press remove
> - I should be on my collegues list
> - I should not see the user in my colleagues list
> - My colleague should not see me in their colleague list

But we're already scheming up a way to improve that situation, so stay tuned as
the story develops.

<!-- #hashrocket -->
