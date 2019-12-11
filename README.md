# Sequential Web-Application Demo

This project demonstrates a web application that is programmed sequentially.
Basically this means its structure mirrors its execution flow.
Such sequential programming of a web application is radically different
from the currently dominant style of web-application programming, but it should
be remembered that the dominant style of programming for many (probably most)
*offline* applications is sequential. The sequential style of programming is
suitable for a wide range of applications and application programmers
because it models a program after the stepwise logic used to complete a
program's goal. The sequential style of programming is the easiest style to
master, all things being equal, because it straightforwardly mirrors our
thinking about what the program must do to fulfil its purpose.
It is interesting to explore the possibility of writing web applications
in a sequential style because the dominant, non-sequential styles
make it difficult to follow execution flow through the program's code
and thus complicate program development, testing, and refactoring.
The discussion thread
["Node.js - A giant step backwards?"](https://news.ycombinator.com/item?id=3510758)
presents problems of a relatively new style of web-application
programming, namely event-driven programming for the web server,
that has become popular in the last few years.

This README briefly explains how a traditional web application and a
web application written in the newer event-driven style (*ala* Node.js) handle
asynchronous events.
Then it points out a working example of an application written for a
continuation-based web server and online resources for learning important
ideas about such applications. Finally it presents a continuation-based
web application that runs entirely in the web browser&mdash;a single-page
web application.


## How Asynchronous Events Are Handled in Web Applications

Any program, simple or complex, that uses I/O (user I/O, disk reads,
disk writes, or transfers of data between computers) must have a means of
synchronizing itself with the completion of that I/O. For desktop applications,
the operating system provides system calls, a scheduler, and library functions
that allow the programmer to structure for his program to mirror
the program's execution flow. For desktop applications, the waiting
for completion of I/O can usually be neatly hidden within some function
like read(), write(), getchar(), etc., but in most web applications written
up to 2019, this mirroring is not possible, due to the stateless nature of
the web. Not even a web application written to run entirely in the
web browser (a single-page web application) can mirror program flow,
due to the event-driven nature of JavaScript in the web browser.

In 2019, web servers can be classified as event-driven (like Node.js) and
non-event-driven (traditional web servers). An event-driven web server
can react to many asynchronous events in real time, without holding up
the main event loop.

Web applications written for an event-driven web server and typical
single-page web applications are structured using callbacks, deferreds,
promises, or some form of continuation-passing style. Thus, their struction
cannot mirror their flow. However, a web application written using *true*
continuations *can* be structured to mirror its flow.


## Traditional Web Applications vs. Web Applications Built upon Server-Side Web Continuations

Very often, web applications interact with the user by building request
pages that pass program state information from web page to web page in
cookies or hidden form fields, something like this Racket Scheme code:

```
	(define (sum query)
	  (build-request-page "First number:" "/one" ""))
	 
	(define (one query)
	  (build-request-page "Second number:"
			      "/two"
			      (cdr (assq 'number query))))
	 
	(define (two query)
	  (let ([n (string->number (cdr (assq 'hidden query)))]
		[m (string->number (cdr (assq 'number query)))])
	    `(html (body "The sum is " ,(number->string (+ m n))))))
	 
	(hash-set! dispatch-table "sum" sum)
	(hash-set! dispatch-table "one" one)
	(hash-set! dispatch-table "two" two)
```

That is the typical, traditional programming style of writing a web application.
Such a style is more complicated and unwieldy than a straightforward style
employing server-side web continuations: 

```
	(define (sum2 query)
	  (define m (get-number "First number:"))
	  (define n (get-number "Second number:"))
	  `(html (body "The sum is " ,(number->string (+ m n)))))
```

Both the web server code and both versions of the application code are
fully described in the section
[Continuations](https://docs.racket-lang.org/more/#%28part._.Continuations%29)
of the page
[More: Systems Programming with Racket](https://docs.racket-lang.org/more/#%28part._.Continuations%29).

The reader is strongly encouraged to
[download Racket](https://download.racket-lang.org/), load the
[the finished Racket Scheme code](https://docs.racket-lang.org/more/step9.txt),
and run the code&mdash;a five- to ten-minute exercise.
The code can be loaded and run in Racket something like this (where 8080 is
the port number to which the server responds and "step9.txt" is the full or
relative pathname of the Racket Scheme code):

```
        $ 𝐫𝐚𝐜𝐤𝐞𝐭
        Welcome to Racket v7.5.
        > (enter! "step9.txt")
        "step9.txt"> (𝐬𝐞𝐫𝐯𝐞 𝟖𝟎𝟖𝟎)
        #<procedure:...webcon/step9.txt:17:2>
        "step9.txt"> 
```

(`$` and `>` are prompts.)
After starting the program, you can use the web application locally
by typing [http://localhost:8080/sum2](http://localhost:8080/sum2)
into your web browser's address bar.
The program asks for and waits for one number, then jumps to a second web page,
where it asks for and gets a second number, then jumps to a third web page,
where it sums the two
numbers. It stores continuations to remember the first and second numbers.
In case the user presses his browser's back button once or twice or retypes the
URL of the second or first page, the program recalls these numbers and
presents them in input forms just like the forms where the user typed them.

Section 5.2 of Christian Queinnec's paper
['Inverting back the inversion of control or, Continuations versus page-centric programming'](https://pages.lip6.fr/Christian.Queinnec/PDF/www.pdf)
describes a very similar web application but does not detail its implementation.


## Client-Side Web Continuations

Server-side web continuations are interesting because of the simplification
they provide for web applications. However, the memory that they consume
can easily become a problem. Since each website user has his own
web browser, it would be convenient to put the continuations in the browser,
naturally scaling the application.
This way, a web application with 10,000 or 1,000,000 users is not burdened
by a heavy price of memory for continuations.
This Git project demonstrates a way to create a web application using
client-side continuations. It includes a submodule,
[jsScheme](https://github.com/tomelam/jsScheme),
which provides the necessary code,
and a test program, ```sequential.txt```.

The test program, apart from supporting functions and macros,
is just this:

```
	(define (test)
	  (with-handlers '((click-handler "foo"))
	    (let ((input (get-input)))
	      (displayln "get-input returned")
	      (displayln input))))
```

The macro ```with-handlers``` sets up any number of
event handlers (```click```, ```mousedown```, ```mouseup```, ```mouseover```,
```timeout```, etc.)
and removes them when execution exits its block.
The function ```get-input``` sets up a continuation that returns
execution to that point only after an event has occurred.

Here are instructions for running the application:

1. Clone the Git repository for
[the demo](https://github.com/tomelam/sequential_web_app_demo).

2. The Git submodule contains a file ```scheme.html```. Open it in a
web browser, either as a file or as a URL. You will see jsScheme's input
textarea and its ```Log``` textarea. To the right of the input textarea are
buttons ```eval```, ```clear input```, etc.

3. Press the ```library``` button. The library will be copied into the input
textarea.

4. Press the ```eval``` button. The library will be added to jsScheme's
work environment and lots of stuff will be printed in the Log.

5. Clear the ```input``` and ```Log```.

6. The cloned repository contains a file ```sequential.txt```. Copy it
into the ```input``` textarea.

7. Press the ```eval``` button. Lots of stuff will be printed in the ```Log```.

8. Clear the ```input``` and ```log```.

9. Type ```(reset (test))``` into the input textarea, then press ```eval```.
9. Type the following into the ```input``` textarea, then press ```eval```.
In the ````Log```` the input will be echoed and ```=> #lambda``` will be
printed.

```
        (𝙙𝙚𝙛𝙞𝙣𝙚 (𝙩𝙚𝙨𝙩)
	  (𝙬𝙞𝙩𝙝-𝙝𝙖𝙣𝙙𝙡𝙚𝙧𝙨 '((𝙘𝙡𝙞𝙘𝙠-𝙝𝙖𝙣𝙙𝙡𝙚𝙧 "𝙨𝙮𝙢𝙗𝙤𝙡𝙨"))
	    (𝙡𝙚𝙩 ((𝙞𝙣𝙥𝙪𝙩 (𝙜𝙚𝙩-𝙞𝙣𝙥𝙪𝙩)))
	      (𝙙𝙞𝙨𝙥𝙡𝙖𝙮𝙡𝙣 "𝙜𝙚𝙩-𝙞𝙣𝙥𝙪𝙩 𝙧𝙚𝙩𝙪𝙧𝙣𝙚𝙙")
	      (𝙙𝙞𝙨𝙥𝙡𝙖𝙮𝙡𝙣 𝙞𝙣𝙥𝙪𝙩))))

	(𝙧𝙚𝙨𝙚𝙩 (𝙩𝙚𝙨𝙩))
```

10. Click on the word 'Input' near the top of the page. (This word is
enclosed in an HTML <div> element having the ID ```foo```,
so it is targeted by the ```click``` handler's
event.) The following will be printed in the ```Log```:
```
	get-input returned
	(click #obj<HTMLTableElement>)
```

11. Click on the table again. Confirm that nothing is added to the `Log`.
This is because the click handler is automatically removed after the
program falls through the `with-handlers` block.
