---
layout: post
title: Radare2 IO plugin tutorial
---

I recently had to write my own IO plugin for Radare2, and as i couldn't find a
good tutorial on the Internet to, i hope this one will help you get going.

You can take a look at the official documentation first:
[Radare2 plugins documentation](https://radare.gitbooks.io/radare2book/content/plugins/plugins.html)

We are going to create an IO plugin called `foo`.

You can download the skeleton from this [URL](https://github.com/Wenzel/radare2-extras/tree/skel/skel)
Or just

~~~
git clone https://github.com/Wenzel/radare2-extras -b skel
~~~

List of the files in `skel/`
~~~
total 24K
-rw-r--r--. 1 wenzel wenzel   53  9 avril 22:27 foo.mk
-rw-r--r--. 1 wenzel wenzel 2,5K  9 avril 22:34 io_foo.c
-rw-r--r--. 1 wenzel wenzel   90  9 avril 22:29 io_foo.h
-rw-r--r--. 1 wenzel wenzel  249  9 avril 22:27 Makefile
-rw-r--r--. 1 wenzel wenzel  158  9 avril 22:16 r2.mk
-rw-r--r--. 1 wenzel wenzel  242 15 avril 20:50 README.md
~~~

## Makefile

`foo.mk` will define the `TARGETS` that we want to build (`io_foo.so`), and
`r2.mk` will define where the plugin should be installed as well as the `CFLAGS`
to link with `radare2`.

## Plugin declaration

Let's have a look at `io_foo.c` and go to the end of the file:

{% highlight C %}
RIOPlugin r_io_plugin_foo = {
    .name = "foo",
    .desc = "IO Foo plugin",
    .license = "LGPL",
    .check = __plugin_open,
    .open = __open,
    .close = __close,
    .lseek = __lseek,
    .read = __read,
    .write = __write,
    .getpid = __getpid,
    .system = __system,
    .isdbg = true,  // # --d flag
};
{% endhighlight %}
This structure defines our plugin `name`, `description` (`r2 -L`), and allows `radare2` call our
own implementation of `read`, `write`, etc...

{% highlight C %}
#ifndef CORELIB
RLibStruct radare_plugin = {
    .type = R_LIB_TYPE_IO,
    .data = &r_io_plugin_foo,
    .version = R2_VERSION
};
#endif
{% endhighlight %}
This one declares our plugin to `radare2`, and set the type to `IO plugin` with
`R_LIB_TYPE_IO`.

## plugin_open

Now the first function you want to implement is `__plugin_open`:
{% highlight C %}
static bool __plugin_open(RIO *io, const char *pathname, bool many) {
    return (strncmp(pathname, URI_PREFIX, strlen(URI_PREFIX)) == 0);
}
{% endhighlight %}
As you can see, it's already implemented. Quite trivial actually. `radare2` will
call this function with a given `pathname` to check if the `prefix` matches when
a URL is opened.

our prefix is:
{% highlight C %}
#define URI_PREFIX "foo://"
{% endhighlight %}
You might want to change that.

## open

Next function too look at is `__open`:
{% highlight C %}
static RIODesc *__open(RIO *io, const char *pathname, int flags, int mode) {
    RIODesc *ret = NULL;
    RIOFoo *rio_foo = NULL;

    printf("%s\n", __func__);

    if (!__plugin_open(io, pathname, 0))
        return ret;

    return r_io_desc_new (io, &r_io_plugin_foo, pathname, flags, mode, rio_foo);
}
{% endhighlight %}

This one is called ` after `__plugin_open`, when `radare2` wants to initialize
our plugin.

you are given the `pathname`: (`foo://something`), the `flags` (`read, write)`, and the
`mode`

In this function you need to initialize your own data structure, (i called it
`RIOFoo`), because it's the only way to maintain some data accross the function
calls (see later).

And when you are done, return a `r_io_desc_new()` to create the `RIODesc *`.

## close

Let's implement `__close` now. 

Simply destroy your own `RIOFoo` and free the
memory !

{% highlight C %}
static int __close(RIODesc *fd) {
    RIOFoo *rio_foo = NULL;

    printf("%s\n", __func__);
    if (!fd || !fd->data)
        return -1;

    rio_foo = fd->data;
    // destroy
    return true;
}
{% endhighlight %}

You can see that we use the `RIODesc->data` to get our `RIOFoo *` structure.

## lseek

This function is called when `radare2` needs to move to another position in the
file, typically with the `s` command.

{% highlight C %}
static ut64 __lseek(RIO *io, RIODesc *fd, ut64 offset, int whence) {
    printf("%s, offset: %llx, io->off: %llx\n", __func__, offset, io->off);

    if (!fd || !fd->data)
        return -1;

    switch (whence) {
    case SEEK_SET:
        io->off = offset;
        break;
    case SEEK_CUR:
        io->off += (int)offset;
        break;
    case SEEK_END:
        io->off = UT64_MAX;
        break;
    }
    return io->off;
}
{% endhighlight %}

I stole this implementation from another plugin.
That's what you want in most of the cases.
To get the details, ask on the Telegram channel.

## read

This one should be easy too, `radare2` wants to read a buffer `buf, at a certain
position, from your `file/whatever`:
{% highlight C %}
static int __read(RIO *io, RIODesc *fd, ut8 *buf, int len) {
    RIOFoo *rio_foo = NULL;

    printf("%s, offset: %llx\n", __func__, io->off);

    if (!fd || !fd->data)
        return -1;

    rio_foo = fd->data;

    return 0;
}
{% endhighlight %}

## write

Same as above, but writing a buffer:
{% highlight C %}
static int __write(RIO *io, RIODesc *fd, const ut8 *buf, int len) {
    printf("%s\n", __func__);

    return 0;
}
{% endhighlight %}

## system

You can implement specific commands in your IO plugin.
When you type `!=command` from the `radare2` shell, this function will be called.

{% highlight C %}
static char *__system(RIO *io, RIODesc *fd, const char *command) {
    printf("%s command: %s\n", __func__, command);
    // io->cb_printf()
    return NULL;
}
{% endhighlight %}

I didn't had to use this, so you will have to implement it by yourself !

And we should have most of the main functions now.

To compile, run `make`, and install with `make install`

you should see your plugin with `r2 -L`
~~~
rwd  windbg   Attach to a KD debugger (windbg://socket) (LGPL3)
rwd  winedbg  Wine-dbg io and debug.io plugin for r2 (MIT)
rw_  zip      Open zip files [apk|ipa|zip|zipall]://[file//path] (BSD)
rwd  foo      IO Foo plugin (LGPL)
~~~

And run it with `r2 foo://something` !
