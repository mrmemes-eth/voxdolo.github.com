---
title: Remote Pairing with Vim and TMux
layout: post
excerpt: |- You can't always pair face-to-face. For those times when you 
  can't, a few sophisticated, yet ubiquitous tools can work small wonders.
---

While it's always best to pair program side-by-side with your pair, it's not
always feasible. Over the last couple of weeks, [Jim Remsik][bigtiger] and I
needed to pair on a new project, but circumstances dictated that he be
elsewhere.

What's a Pair to Do?
--------------------

Keep right on pairing, of course. We just had to figure out how. Our
requirements were:

1. **Collaborative Editing**: the fundament of pairing, both people need to be
   able to type into the same editor.
1. **Access to the Local Server**: it's pretty hard to develop an app you
   can't see.
1. **Ease of Communication**: we need to be able to hear each other,
   preferably seeing each other as well (there's a lot of nonverbal
   communication when pairing).
1. **Light Weight**: the internet connections would be of variable quality, so
   we required something that wouldn't tax small pipes.

We'd previously used was iChat Screen Sharing when we needed to do this at
[Hashrocket][hr]&hellip; it works great for numbers 1 and 2, gets us half way
to number 3, but falls over entirely on number 4. The lag from sending so much
data over the network starts out as merely grating, but quickly crescendos in
abject frustration, tending to make the client of the pairing session little
more than an observer.

Given that, the no-brainer for us was to use [Vim][vim] inside a
terminal-multiplexer that allowed multiple client connections, so we could
attach to the session over SSH. That'd get us "Collaborative Editing", but
what about the other requirements? After a bit of discussion, we decided we
could solve numbers 2 and 4 by forwarding port 3000 of the host machine to the
client attaching via SSH. Being that this was such a lightweight solution, we
opted for [Skype][skype] (our communications weapon of choice) for video chat.

Our initial permutation involved opening a port in our SonicWALL firewall and
manually forwarding the port to a machine inside the firewall with a static
IP. This worked great, after the initial pain of configuring SonicWALL (which
is, in my estimation, a FPOS). The shortcoming here is that we didn't want to
have to open up a port for every station we wanted to remote-pair from (some
days Hashrocket is an elaborate game of musical chairs).  [tpope][tpope] came
to the rescue with his recommendation for using the `ProxyCommand` option for
SSH.

Enough background.

Pairing Down the Configuration
------------------------------

The configuration we used makes several assumptions about your network's
environment:

* At least one machine in your network is available at a public IP
* The externally available machine understands [MDNS][mdns] (e.g. Bonjour)
* The other machines on your network are available via MDNS

That said, it should be relatively easy to modify this to work with a direct
peer-to-peer connection (provided the host has a routable IP address). If
you're so inclined, please contribute in the comments below.


### `.ssh/config`

You'll need an entry for the machine you're tunneling into, ours looks like
this:

{% highlight bash %}
  Host tunnels
    Hostname 0.0.0.0 # this should be an externally routable address
    Port 7676
    User dev
{% endhighlight %}

Now here's the magic:

{% highlight bash %}
  Host *.hashrocket #*
    ProxyCommand ssh -ax tunnels nc `echo %h|sed -e 's/\.hashrocket$/.local/'` %p 2>/dev/null
{% endhighlight %}

Let me attempt to unravel what this is doing. `ProxyCommand` allows you to
specify a command to use to connect to the host machine. In our case, we're
doing a wildcard match on anything at `.hashrocket`, the actual match is then
used to figure out how to route the request inside the network. So in our
case, tunneling to `fry.hashrocket` gets us to our tunnels machine, then we
use netcat to echo the given command to `fry.local`.

Which brings us to the actual forwarding command:

{% highlight console %}
  $ ssh dev@fry.hashrocket -l 3000:localhost:3000
{% endhighlight %}

This drops us into a shell on `fry.local` and establishes the reverse tunnel,
forwarding `localhost:3000` on the host to `localhost:3000` on the client
machine.

### tmux

If you're interested in why we chose [`tmux`][tmux] over [`gnu
screen`][screen], I'll first refer you to the [`tmux` FAQ][tmux_faq] and
subsequently admit that I went with it primarily because it's newer and
shinier. You could easily substitute `screen` for `tmux` here, though my opinion
after a few weeks use is that it's much more intuitive and I won't be making
the move back.

The relevant commands for creating and attaching to a new session follow. On
the host, you'll need to create a session:

{% highlight console %}
  $ tmux new -s mysession
{% endhighlight %}

On the client, you'll need to attach to that session:

{% highlight console %}
  $ tmux attach -t mysession
{% endhighlight %}

Once you're connected, you'll need to run Vim (or some other terminal-friendly
editor, e.g. emacs, sorry TextMate people) in a window or pane. We opted to
run Terminal full screen on an iMac, with two panes arranged vertically; a
shell session in one, Vim in the other and new windows for long-running
processes or utility shells.

### `tmux.conf`

We're Vimtarded, so the following putting the following in `~/.tmux.conf` made
us feel a bit more homey:

{% highlight bash %}
  setw -g mode-keys vi

  bind h select-pane -L
  bind j select-pane -D
  bind k select-pane -U
  bind l select-pane -R
{% endhighlight %}

The initial configuration puts navigation commands in Vim mode (you'll see
this in copy mode, the help screen, etc). The custom bindings below that make
navigating the panes within a window work like navigating splits in Vim.

### `tmux` crash course

The default binding for the command mode is `ctrl-b`. We will refer to it as
<code>&lt;command&gt;</code> hereafter. <code>&lt;command&gt;-d</code> means:
`ctrl-b`, then 'd'.

{% highlight bash %}
  <command>-d # detach from current session
  <command>-" # split window into panes horizontally
  <command>-% # split window into panes vertically
  <command>-o # go to next pane
  <command>-x # close current pane
  <command>-? # display available keybindings
  <command>-c # create a new window
  <command>-n # go to next window
  <command>-p # go to next window
{% endhighlight %}

Pairing is Such Suite Sorrow
----------------------------

So now we're editing text together, we're looking at `localhost` on each end,
we're running some specs, writing some cucumbers, everything's perfect. Until
you need to `save_and_open_page`. You have two options: never make any errors
_or_ move those files somewhere that your pair can see them too.

Since we've yet to be issued our Ninja-Passes, we opted for the latter
and since we're using [Capybara][capy], we did it like so:

In `features/support/env.rb`:

{% highlight ruby %}
  Capybara.save_and_open_page_path = "public/tmp"
{% endhighlight %}

Awesome, now they can just brows to `http://localhost:3000/tmp` and see
Capybara temp pages! That is, if they can guess the name of them, or want to
`ls` the directory, copy the name of the last file and paste it into the
browser.

That *would* work, but how about this instead:

At the bottom of `config/routes.rb`:

{% highlight ruby %}
  if Rails.env.development?
    Some::Application.routes.draw do
      match 'tmp/(:action)', :controller => 'tmp'
    end
  end
{% endhighlight %}

`app/controllers/tmp_controller.rb`:

{% highlight ruby %}
  class TmpController < ApplicationController
    expose(:temp_pages) do
      Dir.glob(File.join('public','tmp','capybara-*'))
    end

    def latest
      render temp_pages.last, :layout => false
    end
  end
{% endhighlight %}

Adding an index view would be trivial if you needed to look at more than just
the latest one[^decent_foot].

The Pair Stare
--------------

As a side-note, we did use Skype to video chat as our primary means of
communication. Jim claims that the best part of it was that I Skyped from my
machine (a 13" MBP, with my iPhone headset) and hosted the `tmux` session on
the iMac right beside it.  This had the unintended side effect of me not
"staring at him" all day long while I coded. It's the little things that make
all the difference :)

Check Out The Pair On This Guy
------------------------------

While my brief stint of remote pairing has come to a close, I've no doubt I'll
need to do it again at some point in the future and am interested in refining
this process. So please, if you can think of a way to improve on or simplify
any of this, leave a note in the comments!

[bigtiger]: http://twitter.com/jremsikjr
[build]: http://windycityrails.org/sessions/#hill
[hr]: http://www.hashrocket.com
[vim]: http://www.vim.org
[skype]: http://www.skype.com
[screen]: http://www.gnu.org/software/screen
[tpope]: http://tpo.pe
[mdns]: http://www.multicastdns.org
[tmux]: http://tmux.sourceforge.net
[tmux_faq]: http://tmux.cvs.sourceforge.net/viewvc/tmux/tmux/FAQ
[capy]: http://github.com/jnicklas/capybara
[^decent_foot]: Curious about that `expose` method? Check out [decent_exposure][de].
[de]: http://github.com/voxdolo/decent_exposure
