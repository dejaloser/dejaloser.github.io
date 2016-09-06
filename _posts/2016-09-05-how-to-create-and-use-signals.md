---
layout: post
title: "How to create and use signals?"
category: prog
tags: gobject
keywords: signal
---

## Simple use of signals

The signal system in GType is pretty complex and flexible: it is possible for its users to connect at runtime any number of callbacks (implemented in any language for which a binding exists) [8] to any signal and to stop the emission of any signal at any state of the signal emission process. This flexibility makes it possible to use GSignal for much more than just emitting signals to multiple clients. 


## Simple use of signals

The most basic use of signals is to implement event notification. For example, given a `ViewerFile` object with a `write` method, a signal could be emitted whenever the file is changed using that method. The code below shows how the user can connect a callback to the "changed" signal. 


file = g_object_new (VIEWER_FILE_TYPE, NULL);

g_signal_connect (file, "changed", (GCallback) changed_event, NULL);

viewer_file_write (file, buffer, strlen (buffer));


The ViewerFile signal is registered in the `class_init` function: 

```
file_signals[CHANGED] = 
  g_signal_newv ("changed",
                 G_TYPE_FROM_CLASS (object_class),
                 G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
                 NULL /* closure */,
                 NULL /* accumulator */,
                 NULL /* accumulator data */,
                 NULL /* C marshaller */,
                 G_TYPE_NONE /* return_type */,
                 0     /* n_params */,
                 NULL  /* param_types */);
```

and the signal is emitted in `viewer_file_write`: 

```
void
viewer_file_write (ViewerFile   *self,
                   const guint8 *buffer,
                   gsize         size)
{
  g_return_if_fail (VIEWER_IS_FILE (self));
  g_return_if_fail (buffer != NULL || size == 0);

  /* First write data. */

  /* Then, notify user of data written. */
  g_signal_emit (self, file_signals[CHANGED], 0 /* details */);
}
```

As shown above, the details parameter can safely be set to zero if no detail needs to be conveyed. For a discussion of what it can be used for, see the section called “The detail argument” 

The C signal marshaller should always be `NULL`, in which case the best marshaller for the given closure type will be chosen by GLib. This may be an internal marshaller specific to the closure type, or `g_cclosure_marshal_generic`, which implements generic conversion of arrays of parameters to C callback invocations. GLib used to require the user to write or generate a type-specific marshaller and pass that, but that has been deprecated in favour of automatic selection of marshallers. 

Note that `g_cclosure_marshal_generic` is slower than non-generic marshallers, so should be avoided for performance critical code. However, performance critical code should rarely be using signals anyway, as emitting a signal blocks on emitting it to all listeners, which has potentially unbounded cost. 


