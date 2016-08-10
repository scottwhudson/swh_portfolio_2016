---
title: Integrating Docs Into Your Rails Workflow
---


The perfect software development workflow is still a mystery in 2016. Software startups are often constantly tweaking their development and deploy processes to ship code as fast as possible, usually taking some hybrid kaizen approach to every aspect of development, from design all the way to deployment. And because of this, it’s really easy to start juggling too many special initiatives and development changes because we seek out the changes that bring the fastest improvements to our process, not necessarily the most impactful. When this happens, we really start to feel the pains of a team that is sacrificing long-term gain for the short-term and winds up being plagued by past decisions. And it is for this reason that fundamental software development practices like product documentation continue to be a challenge, even for larger organizations.

Working for a team of 12 looking to aggressively hire in the short-term forced us to analyze our habits as a development team:

**A.** We had a tendency to ask experienced engineers for information on our codebase instead of taking the time to document the source code itself, which slowed down development and created design anti-patterns that often took more time to fix down the road.

**B.** Attempts at documenting our codebase were short and abandoned early, usually due to us falling behind schedule on other projects, features, or bug fixes.

**C.** The few docs we did have on our products were seldom used, poorly integrated with our workflow, and never continued or improved upon, causing them to stagnate and convincing us that docs weren’t a priority. This in turn caused A., allowing us to put off writing product docs for years.

>If the effort required to maintain the docs exceeded their perceived value to the team, the solution would fail and the effort would become a waste of time.

#### The Solution
While discussing options like jekyll.rb & GitHub wiki, it became clear that maintaining documentation outside of the app was often more work than we really wanted it to be, which was a big red flag for us. If the effort required to maintain the docs exceeded their perceived value to the team, the solution would fail and the effort would become a waste of time.

We started looking at inline documentation generators, ultimately settling on the YARD gem due to its clean syntax and strong community support. But the gem alone wasn’t going to be enough to solve our problem: the process needed to be fully integrated into our workflow to encourage engineers to contribute to it, instead of forcing it through policy.

#### Rails Initializers, FTW

First off, we add the YARD gem to our `Gemfile` as a development dependency & run `bundle`:

```ruby
# Gemfile
group :development do
  gem 'yard'
end
```

Next, we set up the `yard_init.rb` file in the initializers directory, which will cause the following code to execute each time rails is booted.

{% highlight ruby %}
# config/initializers/yardoc_init.rb
#
# Automatically generates the docs if they don't exist and runs yardoc server
# alongside local rails server at http://localhost:8808/.
#
# To manually update your docs build, delete doc/ and reboot the rails server
# or run `yard doc` from the CLI.
if Rails.env.development?
  Thread.new do

    # Only generate docs if they don't exist
    unless Dir.exist?('doc')
      docs = YARD::CLI::Yardoc.new
      docs.run
    end

    # Run the yardoc server
    server = YARD::CLI::Server.new
    server.run
  end
end
{% endhighlight %}

Nothing fancy here. We take advantage of YARD’s CLI class to handle both the document generation and server at the same time. This example only generates docs when the doc folder doesn’t exist, so we’re not regenerating them every time we reboot the server. Any time we do want to regenerate docs, all we need to do is run yard doc or simply rm -r the doc directory.

Lastly, we add the .yardoc and doc directories to our gitignore, so we don’t have to worry about committing them to the repo and keeping them up to date.

NOTE: If you’re using Vagrant, make sure to forward the doc server port in your Vagrantfile

#### Why employ a local docs strategy?
The benefit of this is multifold. First, we’re able to utilize a well-maintained gem to do the heavy lifting for us. Outside of the yardoc_init.rb file, there’s no other infrastructure to maintain. No need to deploy the docs and password protect them or convert them to markdown and shove them in a jekyll site, you have access to your docs whenever you fire up your local rails server. And as long as you’re up-to-date with master, your docs should be as well.

#### The final (and most important) ingredient
While this documentation strategy is clean, minimal, and very effective for teams, this will only work if the team buys into it. Unfortunately, there are no magic tools that can parse ruby source and automatically generate clear, useful documentation. It’s up to those committing the code to make sure docs are being maintained.

The goal with this plan was to make managing documentation as simple as possible, hoping that it would encourage members of the development team to read, use, and ultimately improve the docs. The initial reaction has been great, but we’re really looking forward to the long-term results.
