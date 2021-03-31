Since your repository has issues turned off on github, I'm choosing this way to report ;-)

Feel free to contact me at <github@jmbreuer.net>, or we can use the issue tracker at my fork: https://github.com/jmbreuer/plugin.audio.spotify/issues

# Issue: Spotify Connect does not work

With the plugin installed, Kodi correcly shows up as a Spotify Connect device. However, 
music is not played / there is no reaction when starting playback in a connected Spotify App / application.

## Kodi Log

On every control action (connecting, disconnecting, starting or stopping playback, track skipping), a single stanza is logged:

```
2021-03-31 15:46:27.784 T:8079    DEBUG <general>: 127.0.0.1 - - [31/Mar/2021:15:46:27] "POST /lms/jsonrpc.js HTTP/1.1" 404 1635 "" ""
                                                   
2021-03-31 15:46:27.784 T:8079    DEBUG <general>: .
```

Once again, I find myself wishing for useful "who said that" source identifiers in Kodi's logging. ...

## Analysis

I've seen that the plugin starts a cherrypy HTTP server on port 52308. Poking at it a bit, it looks like it's the thing writing this log line.

Digging further, I've seen that spotty is started to expect an LMS device at localhost:52308 - for future functionality to display information
about what's being played possibly...? Anyway, it seems that the appropriate JSON-RPC service does not exist.

I've found a bug report talking about port overlap, so for testing purposes I turned off all remote control / AirPlay / UPnP services in my Kodi; no change.

For testing, I turned off spotty's LMS integration in `resources/lib/connect_daemon.py:35`, passing only one empty string. Local playback then still works,
Spotify Connect behaves the same (i.e. visible to Spotify Apps, no reaction), but no error messages are logged any more.

I tried to run spotty manually with the arguments as gleaned from `ps`, which causes it to show up as a Spotify Connect target; but I could not get
it to log/trace anything, neither with using `-v` on the command line (which is given by default anyway) nor using the `RUST_LOG` environment variable.

It does print plausible capabilities information using `-x`, but nothing at all without that parameter.

I've tried the 0.12.0 version of spotty next to the normally used one (looks like 0.20.0), it behaves exactly the same except for not understanding
the `--ap-port` parameter.

## Conclusion

I'd love to see, err, hear "more working" Spotify support on Kodi. I hope this information can help towards that.

I'm happy to do further testing and debugging and would love to have some pointers about how spotty/librespot/the 
python part of the plugin interact; which data/control flows where.
And how to get more tracing logs out of spotty / librespot to understand what's going on.
