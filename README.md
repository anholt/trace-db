trace-db is a shared public database of apitrace files and tooling for
Mesa developers.

## Database structure

Traces are stored under traces/appname.  Generally each individual
trace should target some relevant frame: an average scene in the
application, a very expensive scene, or a scene that had some
rendering bug on a driver.

In order to make traces relevant to apitrace-based performance
regression testing, all of the application setup should be in the
first frame of the file.

## Golden reference images

This database is primarily intended to be used with piglit's apitrace
test harness.  Make sure that your driver has some appropriate
categorization (at least driver name, possibly driver name and
hardware family) in piglit.  Known-good frames then are stored as
`traces/<appname>.expected.<drivercategory>.<drawcallnumber>.png`.
They can be generated with:

    apitrace replay -s <appname>.expected.<drivercategory>. appname.trace

(Don't forget the trailing '.')

Frames generated this way should be verified visually against the
reference frames from other drivers.

## Adding a new trace

Begin with:

    apitrace trace myapp

and exit once you've seen what you want to keep.  Identify the frame
you want to keep by:

    apitrace replay -s myapp-frames- myapp.trace

which will produce a set of pngs under myapp-frames-*.png.  Cut all
the frames after the one you want to keep.  Let's say you wanted frame
4:

    apitrace trim myapp.trace -o myapp-end-cut.trace --frames 0-4

Now, see if you can cut the middle frames from the trace:

    apitrace trim myapp-end-cut.trace -o myapp-mid-cut.trace --frames 0,4
    apitrace replay -s myapp-frames-mid- myapp.trace

Compare the myapp-frames-mid-* to myapp-frames-*.png to be sure that
myapp-frames-mid's last frame is still rendering the same thing you
planned on keeping.  Next, import it to the annex:

    mkdir traces/myapp
    
    mv myapp-mid-cut.trace traces/myapp/name-of-frame.trace
    git add traces/myapp/name-of-frame.trace
    
    my myapp-mid-frames-mid-thelastone.png traces/myapp/name-of-frame.expected.png
    git add traces/myapp/name-of-frame.expected.png
    
    cp /usr/share/doc/myapp/copyright traces/myapp/COPYING
    git add traces/myapp/COPYING
    
    git commit -a
