---
layout: post
title:  "Creating avatar using initials in Python with avinit"
date:   2016-08-22 14:01:46 -0300
comments: true
categories: python avatar initials
img:
  user: /assets/2016-08-22/user.png
  da:   /assets/2016-08-22/DA.svg

---

Recently we decided to change our default place holder avatar to use avatar initials.

For instance, if a user is called "Douglas Adams" the avatar should display the user initials as "DA". 
So, basically we wanted to replace this ![anonymous user avatar]({{page.img.user}}){:style="width:25px"}
by that ![avatar using name initials]({{page.img.da}}){:style="width:25px"}.

At first I thought about using [initial.js](http://judelicio.us/initial.js/) to do so as it generates really
nice avatars, but after thinking a little about it I've changed my mind. The reason is simple, our backend
currently serves 2 front-ends (and probably will serve more soon) so we definitely don't want to implement that
avatar logic in each UI.

After deciding to implement the avatars in Python I've started to look for options that are currently
maintained and also supported in Python 3. I've came across with [django-initial-avatars][] and also
with [initials-avatar][] but none of them matched my requirements.

So, why not implement our own lib?! =)

With that in mind I've started checking how each of the 3 libraries (2 Python and 1 Javascript) implemented
and the JS implementation was way simpler than the python versions (and generates nicer avatars).

`Avinit` generates exactly the same SVG avatars than `initial.js` using a very simple template (as follows):

```xml
<svg xmlns="http://www.w3.org/2000/svg" pointer-events="none" width="{width}"
     height="{height}" style="{style}">
  <text text-anchor="middle" y="50%" x="50%" dy="0.35em"
        pointer-events="auto" fill="#ffffff" font-family="{font-family}"
        style="{text-style}">{text}</text>
</svg>
```

`Avinit` is available as open-source (GPL3) in our [github account][avinit] and in PyPI.

To install it using pip just run:

```bash
pip install avinit
```

In the current version (1.0) we implemented two functions (`avinit.get_svg_avatar(text)` and
`avinit.get_avatar_data_url(text)`), both functions receive an text argument, get the first
letter of the first and last words and return an avatar. They difference between them it's
only a matter of format: while `get_svg_avatar` returns an string with the SVG XML
`get_avatar_data_url` returns a formated data URL which can be used directly into the
`<img src="">` tag.

Follows an usage example:

```python
import avinit

avinit.get_svg_avatar('Douglas Adams')
# Returns:
# '<svg xmlns="http://www.w3.org/2000/svg" pointer-events="none" width="46" height="46" style="width: 46px; border-radius: 0px; -moz-border-radius: 0px; background-color: #34495e; height: 46px"> <text text-anchor="middle" y="50%" x="50%" dy="0.35em" pointer-events="auto" fill="#ffffff" font-family="HelveticaNeue-Light,Helvetica Neue Light,Helvetica Neue,Helvetica,Arial,Lucida Grande,sans-serif" style="font-size: 20px; font-weight: 400">DA</text> </svg>'

avinit.get_avatar_data_url('Douglas Adams')
# Returns:
# 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHBvaW50ZXItZXZlbnRzPSJub25lIiB3aWR0aD0iNDYiIGhlaWdodD0iNDYiIHN0eWxlPSJ3aWR0aDogNDZweDsgYm9yZGVyLXJhZGl1czogMHB4OyAtbW96LWJvcmRlci1yYWRpdXM6IDBweDsgYmFja2dyb3VuZC1jb2xvcjogIzM0NDk1ZTsgaGVpZ2h0OiA0NnB4Ij4gPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeT0iNTAlIiB4PSI1MCUiIGR5PSIwLjM1ZW0iIHBvaW50ZXItZXZlbnRzPSJhdXRvIiBmaWxsPSIjZmZmZmZmIiBmb250LWZhbWlseT0iSGVsdmV0aWNhTmV1ZS1MaWdodCxIZWx2ZXRpY2EgTmV1ZSBMaWdodCxIZWx2ZXRpY2EgTmV1ZSxIZWx2ZXRpY2EsQXJpYWwsTHVjaWRhIEdyYW5kZSxzYW5zLXNlcmlmIiBzdHlsZT0iZm9udC1zaXplOiAyMHB4OyBmb250LXdlaWdodDogNDAwIj5EQTwvdGV4dD4gPC9zdmc+'

```

`Avinit` has no dependencies and it's quite simple to use as you can see.
Patches and welcome so please create a fork and submit your Pull Requests!

--
Sérgio Oliveira ([seocam](https://twitter.com/seocam))

[django-initial-avatars]: https://github.com/axiome-oss/django-initial-avatars
[initials-avatar]: https://github.com/Brightcells/initials-avatar/tree/master/initials_avatar
[avinit]: http://github.com/CraveFood/avinit

