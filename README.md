# Anorexic

A thin, lightweight, barebones, mutli-threaded Ruby alternative to Rails (ROR) and Sinatra frameworks... so thin, it's anorexic!

The philosophy is simple - pristine, simple and dedicated gems for each functionality allow for a custom made framework that is exactly the right size during runtime.

Anorexic is a barebones DLS that can run with or without Rack and offers single-port as well as multi-port service for basic and advanced web services alike.

...and since it's all pure Ruby, it's as easy as it gets.

## Installation

install it using:

    $ gem install anorexic

## Framework Usage

to create a new barebones app using the Anorexic framework, run from terminal:

    $ anorexic new appname

That's it, now you have a ready to use basic web server (with some demo code), just run it:

    $ cd appname
    $ ./appname.rb # ( or: anorexic s )

this is a smart framework app that comes very skinny and will happily eat any gem you feed it. it responds extra well to Thin and Haml, which you can enable in it's Gemfile.

now go, in your browser, to: http://localhost:3000/

the default first port for the app is 3000. you can set the first port to listen to by using the `-p ` option (make sure you have permissions for the requested port):

    $ ./appname.rb -p 80

## Barebones Web Service

the app is a simple DSL that deletes all the DLS methods once the server starts running (less clutter while running).

you can run anorexic from your favorite Ruby terminal :) - Anorexic starts the moment you exit the terminal.

this example is basic, useless, but required for every doc out there...

"Hello World!" in 3 lines - try it in irb (exit irb to start server):

		require 'anorexic'
		listen
		route(/.?/) { |req, res| res.body << "Hello World!" }

After you exited irb, the Anorexic server started up. go to http://localhost:3000/ and see it run :)

if more then one `listen` call was made, ports will be sequential (3000, 3001, 3002...) unless explicitly set(`listen 443`).

*btw*: did you notice the catch-all regular-expression? you can write it like this too:

		require 'anorexic'
		listen
		route('*') { |req, res| res.body << "Hello World!" }

Here's a simple web server, complete with SSL, in three (+1) lines of code, serving static pages from the `public` folder::

		require 'anorexic'

		# set up a non-secure service on port 80
		listen 80, file_root: File.expand_path(Dir.pwd, 'public')

		# set up a encrypted service on port 443, works only with some servers (i.e. thin, webrick)
		listen 443, server: 'thin', ssl_self: true, file_root: File.expand_path(Dir.pwd, 'public') 

		shared_route('/people') { |req, res| res.body << "I made this :-)" }

## Anorexic Routes

Routes have paths that tell the application which code to run for every request it recieves. when you set your browser to: `http://www.server.com/the/stuff/they/request?paramaters=params[:paramaters]` , this is the routes path: `/the/stuff/they/request`

As long as Anorexic uses the Anorexic::RackServer class (we could change that, but why would we?), the routes will work the same for all the listening ports.

Anorexic allows your code to choose it's routes dynamically, in the order they are created. like so:

    require 'anorexic'
    listen

    # this route declines to answer
    route('/') { |req, res| res.body << "I Give Up!"; false }

    # this route wins
    route('/') { |req, res| res.body << "I Win!" }

    # this route never sees the light of day
    route('/') { |request, response| response.body << "Help Me!" }

Anorexic accepts Regexp routes as well as string routes and defines a short cut for a catch-all route:

    require 'anorexic'
    listen

    # this route accepts paths that start with a number (i.e.: /nonumber)
    route(/^\/[\d]+[\D]+/) { |req, res| res.body << "Give me more numbers :)" }

    # this route accepts paths that are just numbers (i.e.: /87652)
    route(/^\/[\d]+$/) { |req, res| res.body << "I Love Numbers!" }

    # this route accepts paths that don't have any number (i.e.: /nonumber)
    route(/^\/[\D]+$/) { |req, res| res.body << "Where're my numbers :(" }

    # this route catches everything else.
    route('*') { |request, response| response.body << "Gotcha!" }


## Anorexic Controller classes

One of the best things about the Anorexic is it's ability to take in any class as a controller class and route to the classes methods with special support for RESTful methods (index, show, save, update, before, after):

		require 'pry'
		require 'anorexic'
		require 'thin' # will change the default server to thin automatically.

		class Controller
			def index
				"Hello World!"
			end
			def show
				"You're looking for: #{params[:id]}"
			end
			def debug
				binding.pry
				true
			end
			def delete
				"did you try /#{params["id"]}/?_method=delete"
			end
		end

		listen
		route "/users" , Controller
		route "/" , Controller

Controllers can even be nested (order matters) or have advanced uses that are definitly worth exploring. here's some food for thought:
here's some food for thought:

		require 'pry'
		require 'anorexic'
		require 'thin'

		class ReWriteController
			# using the before filter and regular expressions to make some changes.
			def before
				result = request.path.match /^\/(en|fr)(\/?.*)/
				if result
					params["locale"] = result[1].to_sym
					request.path_info = result[2]
				end
				return false
			end
		end

		class Controller
			def index
				return "Bonjour le monde!" if params[:locale] == :fr
				"Hello World!"
			end
			def show
				return "Vous êtes à la recherche d' : #{params[:id]}" if params[:locale] == :fr
				"You're looking for: #{params[:id]}"
			end
			def debug
				binding.pry
				true
			end
			def delete
				"Mon Dieu! Mon français est mauvais!" if params[:locale] == :fr
				"did you try /#{params["id"]}/?_method=delete"
			end
		end

		listen

		route "*" , ReWriteController

		route /^\/[\d\+\-\*\/\(\)\.]+$/ do |request, response|
			message = (request.params[:locale] == :fr) ? "La solution est" : "My Answer is"
			response.body << "#{message}: #{eval(request.path[1..-1])}"
		end

		route "/users" , Controller

		route "/" , Controller

try:
* http://localhost:3000/
* http://localhost:3000/users
* http://localhost:3000/users/hello
* http://localhost:3000/(5+5*20-15)/9

## Anorexic is hungry for pristine yummy gems

This is the "Pristine chunks" phylosophy.

Our needs are totally different for each project. An XML web service for an iPhone native app is a very different animal then a book-store web app (please, not another book store app...).

Together we can write add-ons and features and beautifuls gems that we will use when (and if) we need them - so our apps are always happy and never overweight!

## What about Ruby on Rails or Sinatra?

I love the Ruby community and I know that we are realy good at writing gems and plug-ins that save a lot of time and code. But we don't need all the plug-ins all the time.

Ruby on Rails became too bloated and big for some projects... It's full of great features that some of them are sometimes used... but at the end of the day, it's HEAVY.

Looking into Sinatra benchmarks on the web showed that Rails and Sinatra frameworks perform on a similar level. The added 'lightness' just wasn't light enough.

Some of us started reverting to pure Rack, and a lot of code kept being written over and over again... Actually, Anorexic is just a smart wrapper to Rack, to make routing and MVC (Model-View-Controller) programming easier.

So sure, you can use Rails or Sinatra, they're great, but we Love to feed Anorexic our code, it just eats it up so nicely.

# Feed the Anorexic framework

The whole of the Anorexic framework philosophy is about community, sharing and feeding the anorexic framework with small and pristine gems.

Please, feel free to contribute, push any changes on the github project and create your own gems to feed the Anorexic open framework.

## Contributing


1. Fork it ( https://github.com/boazsegev/anorexic/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
