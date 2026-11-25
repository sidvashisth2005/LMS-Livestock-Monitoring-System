# Pixy2 camera on the Pi (optional)

The rover normally uses the Pi Camera. A Pixy2 (USB, firmware 3.0.x, `b1ac:f000`) works too, as a stand-in camera.

- The board may be labelled "CMUcam5", but it is a **Pixy2**. The old `charmedlabs/pixy` (Pixy1) library does not work with it; use `charmedlabs/pixy2`.
- Build on the Pi: clone `charmedlabs/pixy2`, patch `swig.dat` (distutils → setuptools), add `video_stop`, `video_resume` and `video_get_raw_frame` to `python_demos/pixy_python_interface.cpp` and `pixy.i`, build, then copy `_pixy*.so` and `pixy.py` into the venv's `site-packages`.
- Install the udev rule `/etc/udev/rules.d/pixy.rules` so the camera works without root.
- Raw frames are 316×208 Bayer at about 60 fps; demosaic with `cv2.COLOR_BayerRG2BGR`. Call `video_stop()` before grabbing raw frames and `video_resume()` before using colour-block tracking.
- Only one process can own the Pixy2 over USB. After stopping a streamer, `init()` may return -7 for a few seconds; retry.
- Image pipeline used by `pi-tools/pixy_stream.py`: brightness 90, demosaic, 3-frame average, grey-world white balance, auto-levels, unsharp mask, 2× upscale, JPEG 85 (about 24 fps). Grab and processing stay in one thread because the SWIG calls hold the GIL.
- Pan-tilt servos: S0 = tilt (higher = up), S1 = pan (higher = right). A sudden servo jump can reset the Pixy2 over USB; a powered USB hub or a separate 5 V supply for the servos helps.
- Control the streamer only with `pi-tools/pixy_stream_ctl.sh start|stop|restart|status`.
