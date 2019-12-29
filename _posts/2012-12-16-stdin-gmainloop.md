---
layout: my-post
title: Reading standard input from GMainLoop
date: 2012-12-16 15:48
---

This time I stumbled upon a challenge: integrate handling of the standard input
data (a very primitive user interaction) with the
[`GMainLoop`](http://developer.gnome.org/glib/stable/glib-The-Main-Event-Loop.html#GMainLoop). I
knew there must be a solution to this problem in a project like
[GLib](http://developer.gnome.org/glib). But as a busy developer I hardly ever
devour entire manuals upfront with all the details. I tend to just skim through
the main sections to get an idea as I prefer the
[RAII](http://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization)
approach: don't touch it if you don't need it. Of course all small pieces of
knowledge amass and at some point it's worth to read the manual in depth to line
up with what's been learned and fill the gaps (these are quite likely huge
gaps).

## The challenge

My real reason for handling user input in the `GMainLoop` was interacting with a
[GStreamer](http://gstreamer.freedesktop.org/) pipeline. This hopefully makes
the story more interesting (introduces some drama). Of course there's a great
*gst-launch-1.0* tool which in many cases is good enough as it allows to build and
run quite complex pipelines. When it comes to interact with a pipeline
(sending/receiving events/messages, reconfiguring the pipeline, setting
element/object properties dynamically), this tool doesn't suffice. One has to
write a GStreamer application.

The other remark is that as a busy developer I prefer to not reinvent the wheel
just to learn something (although sometimes it's reasonable) if there's already
a solution to a problem. Quite often people complicate their lives or even shoot
their own foot by trying the DIY approach. In case of GStreamer they jump
straight into writing a GStreamer application. It's even not that horrible when
they choose Python but it's not hard enough: they use C! And even in C people
are incredibly creative in making their lives miserable. Instead of using
[`gst_pipeline_launch()`](http://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer/html/gstreamer-GstParse.html#gst-parse-launch)
they perform a copy-pasta concerto with the world creation bit by bit. Very
often they would simply get away with `gst-launch-1.0` if they just briefly read
some documentation.

## So why do I need to write a C application?

Why application at all? Can't use `gst-launch-1.0` as I need to interact with the
pipeline. Why not Python? Need to run it on an embedded platform and at the
moment can't afford GObject introspection (additional packages to be
installed). So this is how my story looks like Mr. Jeremy Kyle.

## Ramblings

Very often code examples come with a remark that the error handling is omitted
for code brevity. Fair enough if one dwells on a code snippet and tries to
explain some mysterious API intricacy or some coding phenomenon. I try to follow
a complementary approach: keep code snippets brief but also provide a simple but
complete program that is as accurate as possible when it comes to error handling
(to my best knowledge) and does something useful (at least as a proof of concept
program).

## Let's do it!

The source code for `gmainloop-io-example.c` is
[here](https://github.com/kkonopko/kriscience/blob/master/gmainloop-io-example/gmainloop-io-example.c). Below
I'll show just the relevant excerpts.

Being not a genius and a programmer avoiding _unnecessary_ work I simply
followed a widely advertised skeleton of a GStreamer application with `main()`
initializing the main loop and a GStreamer bus callback. See the [introductory
manual](http://gstreamer.freedesktop.org/data/doc/gstreamer/head/manual/html/index.html}
for more information. For parsing a pipeline description on the command line I
simply stole some code from
[`gst-launch.c`](http://cgit.freedesktop.org/gstreamer/gstreamer/tree/tools/gst-launch.c):

{% highlight C %}
/* make a null-terminated version of argv */
argvn = g_new0 (char *, argc);
memcpy (argvn, argv + 1, sizeof (char *) * (argc - 1));
{
  data.pipeline =
    (GstElement *) gst_parse_launchv ((const gchar **) argvn, &error);
}
g_free (argvn);
{% endhighlight %}

Worth noting is that quite often I see that pipeline parsing/initialization
failure is not handled properly. The
[`gst_pipeline_launch()`](http://gstreamer.freedesktop.org/data/doc/gstreamer/head/gstreamer/html/gstreamer-GstParse.html#gst-parse-launch)
documentation says:

> Please note that you might get a return value that is not NULL even though the
  error is set.

Luckily enough my approach to handle erroneous conditions converged with the one
in `gst-launch.c`:

{% highlight C %}
/* handling pipeline creation failure */
if (!data.pipeline) {
  g_printerr ("ERROR: pipeline could not be constructed: %s\n",
    error ? GST_STR_NULL (error->message) : "(unknown error)");
  goto untergang;
} else if (error) {
  g_printerr ("Erroneous pipeline: %s\n", GST_STR_NULL (error->message));
  goto untergang;
}
{% endhighlight %}

The other thing I've noticed is that quite often GLib source ID variables are
initialized with -1 while they are unsigned. It feels more natural for them to
be initialized with 0 as the
[`g_source_attach()`](http://developer.gnome.org/glib/stable/glib-The-Main-Event-Loop.html#g-source-attach)
documentation promises them to be greater than 0.

## Attaching IO source

My first naive approach was googling for phrases including "IO", "GMainLoop",
"handle" and the like. I ended up reading about
[`GSource`](http://developer.gnome.org/glib/stable/glib-The-Main-Event-Loop.html#GSource)
and actually wrote some source code for my own new GSource class. But that
didn't feel right. This was a good time to have another look at the GLib
documentation. And Eureka! I stumbled across the [IO
Channels](http://developer.gnome.org/glib/stable/glib-IO-Channels.html) section
where I found that
[`g_io_add_watch()`](http://developer.gnome.org/glib/stable/glib-IO-Channels.html#g-io-add-watch)
does all I need. Adding a file descriptor to watch in the GMainLoop is as simple
as this:

{% highlight C %}
GIOChannel *io = NULL;
guint io_watch_id = 0;

/* standard input callback */
io = g_io_channel_unix_new (STDIN_FILENO);
io_watch_id = g_io_add_watch (io, G_IO_IN, io_callback, &data);
g_io_channel_unref (io);
{% endhighlight %}

The `io_callback()` is called whenever there's something to read from the standard
input as I specified `G_IO_IN` as the condition. The callback function reads one
character from the IO channel by calling `g_io_channel_read_chars()`. If it's a
'q', then it quits the main loop. Note that it de-registers itself by returning
FALSE. Otherwise it passes all but the new line characters to a `pipeline_stuff()`
function that is supposed to interpret them and interact with the pipeline. In a
more advanced application the user input would be better structured, e. g. with
some syntax, keywords etc. For this simple program single characters are used
and printed on the console.

{% highlight C %}
static gboolean
io_callback (GIOChannel * io, GIOCondition condition, gpointer data)
{
  gchar in;

  AppData *app_data = (AppData *) data;
  GError *error = NULL;

  switch (g_io_channel_read_chars (io, &in, 1, NULL, &error)) {

    case G_IO_STATUS_NORMAL:
      if ('q' == in) {
        g_main_loop_quit (app_data->loop);
        return FALSE;
      } else if ('\n' != in && !pipeline_stuff (app_data->pipeline, in)) {
        g_warning ("Pipeline stuff failed");
      }

      return TRUE;

    case G_IO_STATUS_ERROR:
      g_printerr ("IO error: %s\n", error->message);
      g_error_free (error);

      return FALSE;

    case G_IO_STATUS_EOF:
      g_warning ("No input data available");
      return TRUE;

    case G_IO_STATUS_AGAIN:
      return TRUE;

    default:
      g_return_val_if_reached (FALSE);
      break;
  }

  return FALSE;
}
{% endhighlight %}

## Build and run

Here's the way I build and run it on my Linux PC:

{% highlight terminal %}
[kris@lenovo-x1 kriscience]$ gcc \
  -o gmainloop-io-example{,.c} \
  $(pkg-config --cflags --libs gstreamer-1.0)
[kris@lenovo-x1 kriscience]$ ./gmainloop-io-example fakesrc ! fakesink
** Message: Running...
eat this
** Message: Pipeline stuff. Received command: e
** Message: Pipeline stuff. Received command: a
** Message: Pipeline stuff. Received command: t
** Message: Pipeline stuff. Received command:  
** Message: Pipeline stuff. Received command: t
** Message: Pipeline stuff. Received command: h
** Message: Pipeline stuff. Received command: i
** Message: Pipeline stuff. Received command: s
q
** Message: Returned, stopping playback
{% endhighlight %}

Note that at the time of this writing I'm using Fedora 17 and I have
GStreamer-1.0 built from the sources and installed in a custom location. My
`PKG_CONFIG_PATH` is set accordingly to reflect that.
