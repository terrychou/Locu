# Locu Help
Locu is an iOS app that helps with opening your daily URLs in a more flexible and faster way. 

[![App Store Badge](https://cdn.rawgit.com/terrychou/locu/master/Download_on_the_App_Store_Badge_en_135x40.svg)](https://itunes.apple.com/us/app/locu/id1224884506?ls=1&mt=8)

Following is the detailed help information for it.

## An example
Say you want to search in Google, with a preferred language for the resulting pages.

In [Google's advanced search page](https://www.google.com/advanced_search), you can do that.

Then the normal way to do it is:
1. open the advanced search page
2. pick the preferred language
3. input the keywords
4. confirm to launch the search

This would be tedious if you have to do them every time.

You can add a command in Locu, set its keyword as “g”, and set its URL as following:

    https://www.google.com/search?q={,-2:Q}&lr=lang_{-1:Q}
    
From now on, when you want to search, say “Greek myth”, and show pages in Simplified Chinese, you can just type in the following invoking text:

    g Greek myth zh-CN
    
Locu will invoke the command you just added, and open the browser with the expected search results.

## Its workflow
As can be seen, Locu turns opening your daily URLs into running invoking text.

Using of Locu involves the following steps:
1. you add and configure URLs into Locu as commands
2. you type in the invoking text
3. Locu filters and shows the matching commands according to the keyword
4. you tap to invoke a command
5. Locu interprets arguments in your text and translates the URL to the final form
6. and Locu opens the translated URL in its handling app

## Command
You configure a command to customize an URL visiting.

A command consists of the following attributes:
* **enabled**
  - whether the command is effective or not (not implemented yet)
* **keyword**
  - the keyword identifying the command
  - required
  - should be unique in Locu
  - no spaces allowed
* **URL**
  - the URL will be opened
  - required
  - contains argument tokens to obtain the respective arguments from the invoking text
* **title**
  - the name of the command
  - required
  - will show as the title in the command entry
* **description**
  - the detailed description of the command
  - argument tokens can also be used to display dynamic information
* **author**
  - the name of the author of this command
* **contact**
  - the contact information of the author

## Invoking text
You express your purpose and pass arguments to URLs by typing in a line of invoking text in Locu.

Here is an example of invoking text:

    kw hello world I\’m here ‘where are you?’
    0    1     2    3     4         5

Its components are separated by spaces or escaping.

3 types of escaping are supported:
- backslash: 
  - the immediate following character will be treated as it is.
- single quotes: 
  - the quoted characters will be treated as one whole component.
- double quotes: 
  - the quoted characters will be treated as one whole component.

After escaping, the components will have their respetive position (tagged below each component in above example):
* position 0 is always the keyword of the invoking text
  - keyword will be used to filter and sort the command list
  - the title and description of a command will be used to match it when there is no exact matching for keyword
* the following positions are arguments
  - arguments will be interpreted and substituted to form the final URL of a command

Therefore, the above example contains 6 components:
* `kw` is the keyword
* `I'm` is a whole component because its single quote has been escaped by the backslash
* `where are you?` is treated as one component thanks to the surrounding single quotes

## Argument tokens
As mentioned above, other components than the keyword in the invoking text will be interpreted as the arguments.

At the URL end, argument tokens are used to express the substitution intension.

In the above Google searching example, `{,-2:Q}` and `{-1:Q}` are two argument tokens. Which will be replaced by relative arguments.

So far, the following argument tokens are supported (the above invoking text is used to help with the explanations):
* {@}
  - all the arguments
  - `hello world I’m here where are you?`
* {*n*}
  - number *n* represents the position of the component
  - negative positions mean ones in the opposite direction
  - {4} means `here`
  - {-2} means the last second component thus equals to {4} in the example
* {*n*,*m*}
  - components in range from poistion *n* through *m*
  - negative positions are supported too
  - if *n* is omitted, it is filled by the beginning
  - if *m* is omitted, it is filled by the end
  - if the range is illegal, it will be empty
  - {1,2} means `hello world`
  - {,-2} means `kw hello world I'm here`
  - {4,} means `here where are you?`
  - {2,1} will be empty since it is invalid
* {*name*}
  - the named argument
  - it represents the argument immediate following the first *name* component 
  - if no matching *name* component, it will be empty
  - the keyword will never be counted as the *name* component
  - {world} means `I'm`
  - {kw} will be empty
* {!*name*}
  - all arguments except for the named argument part
  - the *name* component and its immediate follower will be excluded
  - {!I'm} means `hello world where are you?`
* others
  - will be interpreted as empty strings

When the argument tokens are used in URLs, they need to be percent escaped to ensure the URL's validity.

Locu is smart enough to do this for you (since version 1.2). It will figure out the URL component to which each token belongs, and do the escaping automatically when the command is being launched.

However, you can still indicate the escaping explicitly when it is necessary. The URL escaping attributes can be added within a token by appending them with a colon (`:Q` part in `{,-2:Q}`).

The following attributes are supported:
* H
  - escape for the URL host
  - `http://www.abc{1:H}.com`
* PT 
  - for the URL path
  - `http://www.abc.com/search{2,3:PT}`
* Q
  - for the URL query
  - `http://www.abc.com/search?q={@:Q}`
* F
  - for the URL fragment
  - `http://www.abc.com/search?q=blah#fragment{3:F}`
* U
  - for the URL user name
  - `http://{user:U}@www.abc.com`
* PW
  - for the URL password
  - `http://user:{pwd:PW}@www.abc.com`
* NA
  - do not escape
  - some components, such as scheme and port, inherently don't need escaping
  - `{2:NA}://www.abc.com`
* when omitted, it escapes for query part by default
  - `http://www.abc.com/search?q={@}`
* argument tokens don't need percent escaping when used in description

NOTE: Locu stops doing the automatic escaping whenever any of the tokens contains an explicit escaping attribute.

## Invoking history
Locu will maintain a history list recording every invoking text that has been invoked.

You can re-type a piece of invoking text by just choosing a history entry:
* dragging down the command list will reveal the history list
  - each entry has the invoking text on the left side and relative timestamp on the right
  - releasing will select the highlighted entry
* when revealed lines exceeds 5, the history window will be open
  - scrolling in the window offers a faster history navigation
  - clicking to select one entry and close the window
  - simply swipe up the command list will close the window
  
## Share a command
Of course useful Locu commands can be shared among friends.

And the sharing is very easy because it is operated via [QR code](http://www.qrcode.com/en/):
* to export
  - swipe a command to left to reveal its action buttons
  - choose `More` to invoke its action sheet
  - choose `QR Code` to open its QR code page
  - in this page, you can scan the QR code or export it to other apps
* to import
  - tap the `+` button to invoke the adding menu
  - choose `Capture QR Code` to open the QR code scaning page
  - point the camera to a Locu command QR code will add the command automatically
    - you may need to do some editing before you can import a command when it has some problems

## Other ways to invoke Locu

Besides typing invoking texts within Locu, there are some other ways to utilize it:
* share extension
  - in apps that support text sharing, the text can be shared with Locu as the invoking text
  - you can edit the text before posting, then Locu will try its best to invoke it
* spotlight search
  - you can search Locu's commands with keywords input into the spotlight search
  - choose a command from the list will open Locu and with its keyword already there
* today widget
  - Locu's today widget has 3 buttons
  - `|`: open Locu and start typing
  - `^`: invoke the last entry in history
  - `qp`: try and invoke the text in system clipboard
* URL scheme
  - Locu also supports some custom URL schemes
  - input
    - `locu://input/?text=textToType`
    - Locu will open and have `textToType` input in the search bar
  - run
    - text
      - `locu://run/?text=textToRun`
      - Locu will try and invoke `textToRun` as the invoking text
    - history
      - `locu://run/?history=historyNumber`
      - Locu will try and fetch the history entry at position `historyNumber`, then invoke it
    - clipboard
      - `locu://run/?clipboard=system`
      - Locu will try and invoke the text in system clipboard
  - add
    - `locu://add/?kw=keyword&url=URL&t=title&dsc=description&au=author&co=contact`
    - Locu will try to add a new command with the given parameters
      - please refer to the command attributes for detailed information
      - further editing may be needed if problems happen

## Double spaces
In the search bar, if you type double spaces immediate following the keyword, Locu will replace the current keyword with the topmost command's.

## Contact
If you have any issues, suggestions or requests, please contact [@jixiewu](https://twitter.com/jixiewu)
