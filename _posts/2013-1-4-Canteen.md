---
layout: post
title: Framework Reflections
---

Canteen
=====
I wanted to write a reflection on the work that I have done so far on [Canteen][1], a web framework that I have been working on building from scratch at Hacker School.  I should add up front that not all of the work is mine, I've been working with fellow Hacker Schooler [Angus Burton][2], and he's certainly helped in ways that I couldn't have done myself.

[1]: https://github.com/swoopej/canteen "Canteen"
[2]: https://github.com/angusb "Angus Github"

So the first thing that I learned about is the Web Server Gateway Interface, or WSGI.  I had seen this abbreviation before in the configuration file for Django apps, but it sounded pretty opaque, and I knew that I didn't have to change the default to make my app work, so I occupied myself with other things.

It turns out WSGI is hardly complicated.  It's just a way for the server and the framework or application to communicate, and it's much less complicated form of communication than something like HTTP, which is a protocol that has to be compatible across many platforms.  WSGI only applies to Python servers and Python apps, so the scope is limited and concise.  

The gist of it is that a WSGI server will call a WSGI app with two variables when it receives a request.  One is an [environment dictionary][3] containing information about the request and the server environment, and the other is a function that must be called by the WSGI framework to start the response on the server.  It's really remarkably simple.  So a simple WSGI app using the Python Standard Library WSGI server looks something like this:

[3]: http://www.python.org/dev/peps/pep-0333/#environ-variables "#environ"

    from wsgiref.simple_server import make_server

    class Canteen(object):

        def run_server(self):
        #instantiate the wsgi server
        #receives request, passes to wsgi app
        #send app's response to client
        httpd = make_server(
            'localhost', 
            8051,
            self.wsgi_app #calls the WSGI app
            )

        def wsgi_app(self, environ, start_response):
            '''
            called by the server with the environ dict and the start_response function

            start_response must be called to start the http response.  

            This function will
            return the response body as an iterable.
            '''
            response body = 'yolo'
            status = '200 OK'
            headers = [('Content_Type', 'text/html'),
                ('Content_Length', str(len(response_body)))]
            start_response(status, response_headers)
            return [response_body]

    if __name__ == "__main__":
        app = Canteen()
        app.run_server()

And that's it.  Run that on your local machine, and you will be told that you only live once when you go to port 8051.

The second thing I'll talk about in this post is how we built the interface to our framework, which is nearly identical to that of Flask, another Python framework (side note here:  after digging around in the Flask code for days, it is, in the words of Angus, "quite slick").  

I've built a few small apps in Flask, and it's awesome how quickly you can get an app running.  To add a url to your app in Flask, all it takes is this:

    app = Flask()

    @app.route('/')
    def index():
    	render_template(index.html)

and we managed to copy the same syntax (minus the render_template method for now).  Here's how we did it:

    def add_route(self, path, methods= 'GET'):
        '''decorates a user supplied function by adding path to self.routes'''
        def decorator(f):
            r = Route(path, f, methods)
            self.routes.append(r)
        return decorator 

The decorator creates a new Route object and stores the path, the valid HTTP methods, and the function to be called on the Route object.  That new object is added to the list of Route objects at self.routes, and the decorator function is returned.  When a new HTTP request comes in, framework will go through the Route objects, find the right one, and call the function stored on that objects.  If it doesn't find the right one, you get a 404.

I'll talk about how we search for the correct route object in another post, but I wanted to chime and praise Flask again for how lightweight it is.  While starting an app in Django takes at least five different files, you can import Flask into one file and get quite a bit of functionality out of it.  That's what I hope to reproduce with Canteen.