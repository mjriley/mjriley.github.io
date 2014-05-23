---
layout: post
title: Unity Encoding Issues
---
I felt the need to make a post after spending several hours digging around trying to find a reason why some of my C# Unity GUI scripts would correctly display characters like `é` in labels, while others would display `???`. A popular response seemed to be to convert my scripts to UTF-16, which A) proved difficult to do, and B) didn't seem to solve the problem (it only made the file unreadable in Mono Develop).

The problem is caused if you ever create scripts outside of Unity (like within Mono Develop). Unicode supports the presence of a 'Byte Order Mark' (abbreviated BOM), which is intended to signify the byte order of your characters when you use something like UTF-16 or UTF-32 (Little-Endian vs Big-Endian). In UTF-8 (which is what most people are dealing with), it is redundant, as each character is stored in its own byte, and you can't order a single byte the wrong way. However, it does have the side-effect/hack of being able to identify the document as a 'Unicode' document, which is what Unity/C# is doing here. Scripts created within Unity start with a hidden 3 byte sequence, `0xEFBBBF`, while files created with most editors (apart from Windows Notepad, apparently) don't create anything extra. Most modern editors, as well as tools like 'diff', are trained to ignore that starting sequence, which is why a diff of a Unity-created file and a MonoDevelop-created file won't actually show any differences. It's also why you won't see garbage characters at the start of whatever file you're editing.

Unity, specifically the Microsoft C# and Mono C# compilers, will not identify text files as UTF-8 encoded *unless* the file begins with that byte sequence. Presumably, this is because the C# compiler supports many 'codepage' options; if no codepage is explicitly specified, it falls back on the default system codepage, which, at least in my case, was ASCII (you can find your own by accessing the `System.Text.Encoding.Default.EncodingName` property). The [Remarks Section here] (http://msdn.microsoft.com/en-us/library/system.text.encoding.default.aspx) has some more detailed information on why the compiler relies on those 3 bytes for UTF-8.

Anyway, that's the reason.. so what can you do about it if you don't enjoy creating your scripts within Unity? A few solutions, depending on your editor:

* If you use Vim, set the :bomb option (mentioned [here](http://vim.1045645.n5.nabble.com/How-to-display-and-remove-BOM-in-utf-8-encoded-file-td4681708.html)).
* If you use Emacs, check out [this](http://superuser.com/questions/41254/make-emacs-not-remove-the-bom-from-xml-files).
* If you use Mono Develop -- save your script via 'Save As', selecting 'UTF-8' under the encoding option.

If you want a more fire-and-forget option, you can actually set the C# compiler options that Unity uses by creating a special file, outlined [here](https://docs.unity3d.com/Documentation/Manual/PlatformDependentCompilation.html) all the way under 'Global Custom Defines'. Basically, you need to create a file called 'smcs.rsp' under your Assets directory, and within that file, you can specify any option listed [here](http://msdn.microsoft.com/en-us/library/6ds95cz0.aspx). Specifically, you'll need to create a line with `-codepage:utf8`. Keep in mind this will override the encoding used for ALL of your scripts.

Finally, if you care about the background for the Unicode BOM process, you can read up on it [here](http://tools.ietf.org/html/rfc3629#page-6).
