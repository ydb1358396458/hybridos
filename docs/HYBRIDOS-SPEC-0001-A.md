# HybridOS View Markup Language

HybridOS Specification 0001-A  
Author: Vincent Wei  
Category: App Framework  
Date: November 2018  
Status: Proposal

*Copyright Notice*

Copyright (C) 2018, 2019 [FMSoft Technologies]  
All Rights Reserved.

## Introduction

A new app framework defines a new operating system, while the
new app framework first needs a new GUI programming model.

In GUI programming model, the event-driven programming
model has gained popularity. Event-driven programming is structured
around the concept of decoupled relationships between event producers
and event consumers.

Currently, almost all mature GUI support systems are event driven.
In order to facilitate development, these GUI support systems, including
Gtk+, Qt, MiniGUI, Windows, Android, iOS basically adopt concepts
such as Toolkit, Widget, or Control. The developer puts various
widgets on a window or an activity, and manipulates the widgets or
handles the events generated by them.

However, the traditional widget-based programming mechanism does not
provide a good solution for decoupling data and rendering relationships.

On the other hand, the development of Web technologies,
especially the development of HTML5 and CSS 3, makes webpages
more interactive, and provides a simple, easy to use and flexible
coding and implementation mechanism, especially on decoupling
the data and rendering relationships.

The designers of HybridOS believe that the client GUI programming
mechanism should learn from Web technologies. Furthermore, we
believe that the many benefits of web client programming are not
derived from the JavaScript language, but rather the underlying
structured document description mechanism (DOM) and the CSS-based
style definition mechanism.

Therefore, HybridOS's GUI programming model no longer adheres to
the traditional Toolkit/Widget mechanism; we choose to extend HTML5 and
introduce the support of CSS 3. When programming is required, HybridOS
provides the JavaScript programming interface same as Web browser
for high-end device configurations and a C++ programming interface for
low-end device configurations.

The key features follow:

1. The developer always use HVML (HybridOS View Markup Language, which
   defines a few new tags based on HTML 5.3) and CSS 3 to define
   the structure, styles, and layout of the elements and GUI.

2. If the hardware has enough computing power (at lease 64MB RAM and 600MHz CPU),
   the developer can write the device apps in JavaScript language. Or,
   the developer falls back to write the device apps in C++ language.

3. For the device apps written in JavaScript language, HybridOS provides a
   user agent like a Web browser, which contains the V8 JS engine and the
   HVML renderer. We name the user agent as 'HybridOS App Engine'
   (`HAE` for short).

4. When you write the device apps in C++ language, the HVML documents
   and CSS definitions can be interpreted by some ready-to-use tools
   to one or more C++ source files. The developer can manipulate all
   DOM elements, their attributes, styles, and content like JavaScript
   does. You can even convert existed HTML webpages to the C++ source files.
   In this way, we can use the same UI description files for
   device apps, client apps, and even web apps.

5. HAE will be cross-platform, one device app written in JavaScript
   can run on any operating system (Linux/Windows/macOS/Android/iOS)
   directly.

We think that this is a universal solution for the GUI programming framework,
and never be out of date.

This specification describes the extended HTML 5.3 tags (HVML tags) supported by
HybridOS, and the basic method to define a HybridOS app.

## The HVML Tags

`HVML` means HybridOS View Markup Language, which defines a few new tags based on
[HTML 5.3].

The HVML tag `view` represents a complex widget which can not be described and
rendered easily by using the standard HTML 5.3 tags and CSS, for example, a chart,
a calendar, a meter panel, and so on.

As we know, HTML 5.3 introduced some new tags especially the ones for
interaction, such as `progress`, `meter`, `details`, `summary`,
and `dialog`. HVML provides well support for these tags.

However, if we want to show a complex widget, we either use a plugin or
use the `canvas` to render them by using script. In order to reduce the
development efforts and improve the performance, we introduce the `view`
tag for HVML.

For example, to define a meter panel, we use the following HVML tag `view`:

    <view type="meter" class="CLASS" name="NAME" id="IDENTIFIER">
        <param name="NAME" value="VALUE">
        <param name="NAME" value="VALUE">
    </view>

HTML 5.3 also introduced the `template` tag. A template is a virtual element,
which can be used to generate other real elements by substituting some
attributes and inserting them to the DOM tree.

However, you still need to write a piece of script code to clone the
template in HTML. But in HVML, you can use the following new tags to
clone a template element without the script code:

* `iteration`: define an iteration.
* `substitution` and `condition`: define conditional substitution.

The tags above use the intent data defined in the head element of a
HVML document:

* `intent`: like `style` or `script` tags, this tag represents the intent
data passed to the document. We use the `textContent` of an intent
element to define the real intent data, but the intent data must be in
JSON format. The `name` attribute of this tag gives the name for the
intent data, and the optional boolean attribute `once` tells the user
agent whether to destroy the intent data after used the data.

For example:

    <!DOCTYPE hvml>
    <hvml>
        <head>
            <intent name="users" once>
                [
                    { "id": 1, "avatar": "/img/avatars/1.png", "Tom" },
                    { "id": 2, "avatar": "/img/avatars/2.png", "Jerry" }
                ]
            </intent>
            <intent name="global">
                { "locale" : "zh_CN" }
            </intent>
        </head>

        <body>

            <template id="footer-cn">
                <p><a href="http://www.baidu.com">Baidu</a></p>
            </template>

            <template id="footer-tw">
                <p><a href="http://www.bing.com">Bing</a></p>
            </template>

            <template id="footer-def">
                <p><a href="http://www.google.com">Google</a></p>
            </template>

            <template id="user-item">
                <li class="user-item" data-key="id" data-for="attribute::data-user-id">
                    <img data-key="avatar" data-for="attribute::src">
                    <span data-key="name" data-for="textContent"></span>
                </li>
            </template>

            <template id="no-user">
                <li>
                    <img src="def-avatar.png" />
                    <p>Sign Up for the first user!</p>
                </li>
            </template>

            <header>
                <h1>User List</h1>
            </header>

            <main>
                <ul>
                    <iteration template="user-item" intent="users" alt="no-user" />
                </ul>
            </main>

            <footer class="footer">
                <substitution intent="global">
                    <condition template="footer-cn" data-key="locale" match-value="zh_CN" />
                    <condition template="footer-tw" data-key="locale" match-value="zh_TW" />
                    <condition template="footer-def" />
                </substitution>
            </footer>
        </body>

    </hvml>

## A Sample

In this section, we provide a sample to show how the HybridOS View Markup Language works.
In this sample, we show a user list with avatars and user names. When you clicked on a
user item, the app will show the detailed information of the user.

### Define activities

The following markup statements define the user list activity:

    <!DOCTYPE hvml>
    <!-- The user list activity -->
    <hvml>
    <head>
        <meta name="activity" content="act-user-list" />
        <intent name="users"  once>
            [
                { "id": 1, "avatar": "/imgs/avatar/1.png", "Tom" },
                { "id": 2, "avatar": "/imgs/avatar/2.png", "Jerry" }
            ]
        </intent>
        <intent name="global">
            { "locale" : "zh_CN" }
        </intent>
    </head>

    <body>

        <!-- define templates for future use -->
        <template id="user-item">
            <li class="user-item" data-key="id" data-for="attribute::data-user-id">
                <img data-key="avatar" data-for="attribute::src">
                <span data-key="name" data-for="textContent"></span>
            </li>
        </template>

        <template id="footer-cn">
            <p><a href="http://www.baidu.com">Baidu</a></p>
        </template>

        <template id="footer-tw">
            <p><a href="http://www.bing.com">Bing</a></p>
        </template>

        <template id="footer-def">
            <p><a href="http://www.google.com">Google</a></p>
        </template>

        <header>
            <h1 name="theHeader" content="$STRID_TITLE" class="panel-header"></h1>
        </header>

        <main>
            <ul>
                <iteration template="user-item" intent="users" />
            </ul>
        </main>

        <footer class="footer">
            <substitution metadata="global">
                <condition template="footer-cn"  data-key="locale" match-value="zh_CN">
                <condition template="footer-tw"  data-key="locale" match-value="zh_TW">
                <condition template="footer-def" default>
            </substitution>
            <p content="$STRID_COPYING" class="footer-copying"></p>
        </footer>

        <script>
            /* bind the click event of userItem */
            $(".user-item").on ('click', function ($item) {
                /* launcher userInfo activity */
                launchActivity ("act-user-info", {id: $item.id});
            });
        </script>
    </body>
    </html>

The following markup statements define the user information activity:

    <!DOCTYPE hvml>
    <!-- The user information activity -->
    <hvml>
    <head>
        <meta name="activity"   content="act-user-info" />
        <intent name="userInfo"  once>
            <JSON DATA>
        </intent>
    </head>
    <body>

    </body>
    </hvml>

## L10N

Different from webpages, HVML handles the L10N text as an asset of an app. 
HAE (the HVML user agent) loads the L10N file in JSON and translate the
localization text according to the identifier.

If you specify a content of `text` with a string started with `$`, the
string will be treated as the identifier of a text instead of the literal text.
To escape from the rule, use `\$` for the prefix.

The content of a L10N text file will look like:

    {
        "app": "firstSample",
        "locale": "zh_CN",
        "STRID_TITLE": "HybridOS 的第一个示例应用",
        "STRID_COPYRING": "版权所有 (C) 2018 飞漫软件",
        "Welcome to the world of <em>HybridOS</em>!": "欢迎来到 <em>HybridOS</em> 的世界！",
    }

Obviously, `app` and `locale` are reserved keywords for L10N text file.

### Define app

As you can see from the sample code above, we need to define an app object as the
entry of the app and control the different activities.

An app can be defined by using the following markup statements:

    <!DOCTYPE hvml>
    <hvml name="firstSample" lang="en">
        <!-- define the assets of the app, such as the activities, images, L10N text, CSS, and so on -->
        <head>
            <meta name="activities.act-user-list"   content="userlist.html" default />
            <meta name="activities.userInfo"        content="userinfo.html" />

            <!-- the links to the localization translation files */
            <link rel="localtext" type="text/json" hreflang="zh_CN"
                    href="/firstSample/assets/messages/zh_CN.json" />
            <link rel="localtext" type="text/json" hreflang="zh_TW"
                    href="/firstSample/assets/messages/zh_TW.json" />
            <link rel="localtext" type="text/json" hreflang="en_US"
                    href="/firstSample/assets/messages/en_US.json" />

            <link rel="stylesheet" type="text/css" href="/firstSample/assets/default.css" />
        </head>
    </hvml>

Obviously, if you use the method above to define the app, you need prepare three files:

* `index.hvml`: the app;
* `userlist.hvml`: the `act-user-list` activity;
* `userinfo.hvml`: the `act-user-info` activity.

## HVML and CSS

HybridOS uses CSS to define the style of all view types. You can apply your own
CSS to any HVML element as well.

## The HTML5 Technologies Supported

The following HTML5 Technologies will be supported by HVML in the first official release:

* canvas and canvas object.
* WebStorage (including localStorage and sessionStorage)
* WebSocket
* Geolocation

The following HTML5 Technologies will be supported in the future releases:

* SVG
* MathML
* Audio
* Video

## Extended JavaScript Objects

HybridOS will provide the following extended JavaScript Objects for app:

* SQLite for local database.
* hBus for communication with the local system services.
* MQTT for communication with the cloud and clients.

## Relationship with C++ App Framework

Under C++ language, we use the HybridOS Foundation C++ Class Library
(`HFCL` for short, see [HybridOS Foundation Class Library]).

HFCL is derived a C++ GUI framework called mGNGUX, which is based on MiniGUI.
We re-designed HFCL to make this library can be ported easily to other
host operating system, for example, Windows or macOS.

Hybrid App Engine is developed based on HFCL.

---

_The following words are deprecated..._

Otherwise, if your app is a simple one, you can organize your code in the following manner
to use only one file:

    <!DOCTYPE hvml>
    <hvml name="firstSample" lang="en">
        <head>
            <meta name="activity:act-user-list" content="userlist.html" />
            <meta name="activity:act-user-info" content="userinfo.html" />
            <meta name="img:defAvatar" content="/firstSample/assets/default.css" />

            <link rel="localtext" type="text/json" href="/firstSample/assets/messages/zh_CN.json" />
            <link rel="stylesheet" type="text/css" href="/firstSample/assets/default.css" />
        </head>

        <activity id="act-user-list" hbd-scope="firstSample">
            ...
        </activity>

        <activity id="act-user-info" hbd-scope="firstSample">
            ...
        </activity>

        <script>
            ...
        </script>
    </hvml>

We use a HTML document to define an activity. However, different from HTML,
we can define multiple view clients for one activity, and at the same time,
only one view client can be kept active. The other view clients are virtual
and invisible from the users.

In a `view` or `template`, we use the standard sectioning and/or heading
content tags of HTML5 to define the structure of the view:

* `article`, `aside`, `nav`, `section`
* `header`, `footer`, `main`, `div`
* `ul`, `ol`, `dl`
* `address`
* `blockquote`
* `figure`
* `details`

and use the following tags to define the text paragraphs:

* `h1`, `h2`, `h3`, `h4`, `h5`, `h6`
* `p`, `prev`
* `li`, `dt`, `dd`
* `summary`
* `figcaption`

In a text paragraph, we use the phrasing content elements (text-level
elements and embedded elements) of HTML5 to define the content.
Therefore, you can use the following HTML5 tags:

* The text-level elements:
  * `a`, `abbr`, `b`, `bdi`, `bdo`, `br`, `cite`, `code`, `data`, `datalist`
  * `del`, `dfn`, `em`, `i`, `ins`, `kbd`, `label`, `mark`,
  * `output`, `q`, `ruby`, `s`, `samp`, `small`, `span`
  * `strong`, `sub`, `sup`, `time`, `u`, `var`, `wbr`

* The embedded elements:
  * `img`, `button`, `input`, `math`, `meter`, `picture`, `progress`
  * `math`, `svg`,
  * `audio`, `video`

### Tag properties

1. `class`: A global HTML5 property, which specifies the class of the view.

1. `name`: A global HTML5 property, which specifies the name of an app,
an activity, a view, or a template, Generally, the value of this property
will be a variable which you can refer to in your JavaScript or C++ code.

1. `id`: A global HTML5 property, which specifies the identifier of an app,
an activity, a view, or a template, Generally, the value of this property
will be a variable which you can refer to in your JavaScript or C++ code.

1. `content`: A global HVML property, which specifies the content of an
element.

### Differences between HVML and HTML

1. HVML is defined for user interfaces not documents

1. Easy to localize

In HTML5, we define a text paragraph like this:

    <p>
        Welcome to the world of <em>HybridOS</em>!
    </p>

When we want to localize the paragraph for other locale, we generally
generate a different webpage for the locale.

However, in HVML, we define a text paragraph in the following way:

    <p content="$Welcome to the world of <em>HybridOS</em>!">
    </p>

So a JavaScript or C++ code can easily translate the content into other
locale by using a GNU message file or a JSON table.

## Embed App into HTML5 Pages

It is possible to embed a HybridOS app into an existed HTML5 page:

    <!DOCTYPE html>
    <html>
        <head>
            <meta name="viewport"
                content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no"/>
            <meta name="format-detection" content="telephone=yes"/>
            <meta name="apple-touch-fullscreen" content="yes"/>

            <title>The first sample of HybridOS embedded in a HTML5 page</title>

            <link rel="stylesheet" type="text/css" href="//hybridos.fmsoft.cn/js/hybridos.css" />
            <script type="text/javascript" src="//hybridos.fmsoft.cn/js/hybridos.js"></script>
        </head>

        <body>

            <header>
                <h1>The First Sample of HybridOS</h1>
            </header>

            <p>This page shows howto embedded a HybridOS app in a HTML5 page.</p>

            <hvapp id="firstSample">
                <assets>
                    ...
                </assets>

                <activity id="act-user-list" hbd-scope="firstSample">
                    ...
                </activity>

                <activity id="act-user-info" hbd-scope="firstSample">
                    ...
                </activity>

                <script>
                    ...
                </script>
            </hvapp>

            <footer>
                ...
            </footer>
        </body>
    </html>

Note that we import `hybridos.css` and `hybridos.js` in the head of the HTML5 page.

_The above words are deprecated..._

[Beijing FMSoft Technologies Co., Ltd.]: https://www.fmsoft.cn
[FMSoft Technologies]: https://www.fmsoft.cn
[HybridOS Official Site]: https://hybrid.fmsoft.cn

[HybridOS Architecture]: https://github.com/VincentWei/hybridos/blob/master/docs/HYBRIDOS-SPEC-0000.md
[HybridOS App Framework]: https://github.com/VincentWei/hybridos/blob/master/docs/HYBRIDOS-SPEC-0001.md
[HybridOS View Markup Language]: https://github.com/VincentWei/hybridos/blob/master/docs/HYBRIDOS-SPEC-0001-A.md
[HybridOS Foundation Class Library]: https://github.com/VincentWei/hybridos/blob/master/docs/HYBRIDOS-SPEC-0002.md
[HybridOS Security Design]: https://github.com/VincentWei/hybridos/blob/master/docs/HYBRIDOS-SPEC-0003.md
[HybridOS Device Simulation Environment]: https://github.com/VincentWei/hybridos/blob/master/docs/HYBRIDOS-SPEC-0004.md
[HybridOS Code and Development Convention]: https://github.com/VincentWei/hybridos/blob/master/docs/HYBRIDOS-SPEC-0005.md

[HTML 5.3]: https://www.w3.org/TR/html53/