- [Commands](Commands.md)



## PictureFrame powered by pi3d

![picframe logo](https://github.com/helgeerbe/picframe/wiki/images/Picframe_Logo.png)

- [PictureFrame powered by pi3d](#pictureframe-powered-by-pi3d)
- [What Is PictureFrame?](#what-is-pictureframe)
- [History of PictureFrame](#history-of-pictureframe)
- [Highlights of PictureFrame](#highlights-of-pictureframe)
- [Documentation](#documentation)
- [Acknowledgement](#acknowledgement)

## What Is PictureFrame?

This is a viewer for a raspberry powered picture frame. For remote control it provides an automatic integration into [Home Assistant](https://www.home-assistant.io/) via MQTT discovery.

- https://github.com/helgeerbe/picframe
- Paddy Gaunt, Jeff Godfrey, Helge Erbe
- Licence: MIT
- Tested on rasberry 3B+/4, Ubuntu 20.10 and Python 3.7

## History of PictureFrame

When I started 2019 my DIY project building a raspberry powered digital picture frame I came across Wolfgang's website [www.thedigitalpictureframe.com](https://www.thedigitalpictureframe.com/). I ran my frame with the [pi3d PictureFrame2020.py](https://github.com/pi3d/pi3d_demos) viewer, but always missed a more deeply integration to my smart home server running [Home Assistant](https://www.home-assistant.io/).As my personel corona project I decided to rewrite the viewer to my needs. Hoping  someone can make use of it.


## Highlights of PictureFrame

- Viewer
  - blend effects
  - auto mat generation
  - photo metadata overlays (title, location, date, ...)
  - live clock
  - automatic pairing of portrait images
  - keyboard, mouse and touch screen support
- Filter by
  - IPTC tags
  - location
  - directories
  - date
- Remote Control
  - control interface for mqtt, http(s)
  - tun on/off display
  - next/prev/pause image
  - shuffle play
  - toggle metadata overlays
  - toggle clock visibility
  - retrieve image meta info (exif, IPTC)

## Documentation

[Full documentation can be found at the project's wiki](https://github.com/helgeerbe/picframe/wiki).

Please note that PictureFrame may change significantly during its development.
Bug reports, comments, feature requests and fixes are most welcome!

To find out what's new or improved have a look at the [changelog](https://github.com/helgeerbe/picframe/wiki/Changelog).

## Acknowledgement

[glenvorel](https://github.com/glenvorel) Thanks for the new keyboard, mouse and touch screen support.

Many Thanks to Wolfgang [www.thedigitalpictureframe.com](https://www.thedigitalpictureframe.com/) for your inspiring work. 

A special Thank to Paddy Gaunt one of the authors of the [pi3d](https://github.com/pi3d/pi3d_demos) project. You are doing a great job!

Last but no least a big Thank You to Jeff Godfrey. Your auto mat feature and database driven cache is an outstanding piece of code.
