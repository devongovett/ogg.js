ogv.js
======

libogg, libvorbis, and libtheora compiled to JavaScript with Emscripten.


## Current status

A demo is included which runs some video output in the browser; you can
search within a list of Wikimedia Commons 'Media of the Day'. It will
appear under build/demo/

See a web copy of the demo at https://brionv.com/misc/ogv.js/demo/

* streaming: buggy, some buffering problems
* color: yes
* audio: buggy & limited (no IE or iOS)
* background threading: no


## Goals

Long-form goal is to create a drop-in replacement for the HTML5 video and audio tags which can be used for basic playback of Ogg Theora and Vorbis media on browsers that don't support Ogg or WebM natively.

(Note that a more user-friendly solution in most cases is to provide media in both open and MPEG-LA formats, if you're not averse to using patent-encumbered formats. This will use much less CPU and battery than performing JavaScript decoding!)


Short-ish clips of a few seconds to at most a few minutes at SD resolution or below are the primary target media. This system should really not be used for full-length TV or movies, as it's going to eat battery horribly due to sustained high CPU usage.


The primary target browsers are:
* Safari 6+ on Mac OS X
* Internet Explorer 10+ on Windows

Future targets (currently not acceptable performance):
* Safari on iOS 6+
* Internet Explorer 10+ on Windows RT

(Note that Windows and Mac OS X can support Ogg and WebM by installing codecs or alternate browsers with built-in support, but this is not possible on iOS or Windows RT.)

Testing browsers (these support .ogv natively):
* Firefox 24
* Chrome 30


## Performance

Early versions have only been spot-checked with a couple of small sample files on a few devices, but for SD-or-less resolution basic decoding speed seems adequate on desktop. Newer mobile devices seem to handle at least low-res files, but much more tuning and measurement is needed.

*Target browsers*

On Mac OS X, Safari 6.1 and 7 perform much better than Safari 6.0; however 6.1/7 have not yet been generally released. Have patience! Note that Safari seems to disable the JIT when the developer console is open, so beware when debugging.

IE 10 and IE 11 on Windows 8 and 8.1 perform pretty well. Older versions of IE are not supported at all.


*Future targets*

See [device notes](https://github.com/brion/ogv.js/wiki/Device-notes) for testing status.

On iOS, Safari performs significantly better than Chrome or other alternative browsers that are unable to enable the JIT due to iOS limitations on third-party developers. However, I have not yet gotten acceptable performance on non-tiny files even on the latest iOS 7 Safari. Needs more testing on the iPhone 5S however!

IE 11 on Windows RT 8.1 on an original Surface RT tablet does not perform very well.

In both cases, a native application looms as a possibly better alternative. If installed, a native app could definitely be launched from web content... Need to look into how easy it is to detect presence of apps from web, however; if it's not possible to detect it may need a UX workaround to prompt the user.


*Test browsers*

Firefox 24 performs best using asm.js optimizations -- unfortunately due to limitations in the JS engine this currently only works on the first video playthrough. Reload the page to force a video to re-run at high speed.

Chrome 30 performs pretty well, but not quite as snappy as Firefox's asm.js mode.

It would also be good to compare performance of Theora vs VP8/VP9 decoders.

YCbCr->RGB conversion could be done in WebGL on supporting browsers (IE 11, Chrome, Firefox), if that makes a measurable difference.


## Difficulties

*Threading*

Currently the video and audio codecs run on the UI thread, which can make the UI jumpy and the audio crackly.

WebWorkers will be used to background the decoder as a subprocess, sending video frames and audio data back to the parent web page for output. This should be supported by all target and test browsers.

It may not be possible to split up the codec work over multiple workers, but this will at least get us off the UI thread and make the page more responsive during playback.


*Streaming*

In IE 10, the (MS-prefixed) Stream/StreamReader interface is used to read data on demand into ArrayBuffer objects.

In Firefox, the 'moz-chunked-array' responseType on XHR is used to stream data, however there is no flow control so the file will buffer into memory as fast as possible, then drain over time.

Currently in Safari and Chrome, streaming is done by using a 'binary string' read. This has no flow control so will buffer into memory as fast as possible. This will also buffer up to twice the size of the total file in memory for the entire lifetime of the player, which is wasteful but there doesn't seem to be a way around it without dividing up into subrange requests.


*Seeking*

Seeking is tough. Need to do some research:
* how to determine file length in time
* how to estimate position in file to seek to based on time target
* how to reinitialize the decoder context after seeking

Jumping to a new position in the file that hasn't yet been buffered could be accomplished using partial-content HTTP requests ('Range' header), but this requires CORS header adjustment on the server side.


*Audio output*

Safari and Chrome support the W3C Web Audio API (with 'webkit' prefix). Explicit synchronization is not yet performed, so audio and video drift out of sync over time.

Note that audio fails on iOS as web audio must be started in an event handler for a user action.

Audio is blacklisted on Safari 6.0 due to a possible bug in the JavaScript VM or JIT compiler -- Vorbis audio *decoding* hangs the CPU unless the debug console is open (which makes things run rreeaallyy ssllooww). Safari 6.1 and 7 work just fine and are not blacklisted.

Firefox supports Web Audio API with an optional about:config switch.

Unfortunately IE doesn't support Web Audio yet... Audio playback on IE may need to use a shim via the Flash plugin (which is bundled), which may make sync more difficult as there's another layer between our JS code and the output.


## Building

1. Install [Emscripten](https://github.com/kripken/emscripten/wiki/Tutorial).
2. `git submodule update --init`
3. Install [importer](https://github.com/devongovett/importer) with `npm install importer -g`.
4. Run `make` to configure and build libogg, libvorbis, libtheora, and the C wrapper. Run this again whenever you make changes to the C wrapper or a new version of libogg is released.

See a sample web page in build/demo/


## License

libogg, libvorbis, and libtheora are available under their respective licenses, and the JavaScript and C wrapper code in this repo is licensed under MIT.

Based on build scripts from https://github.com/devongovett/ogg.js
