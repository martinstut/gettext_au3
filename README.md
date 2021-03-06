# gettext_au3 - Internationalization of AU3 Applications by Something Similar to GNU Gettext

For a general overview of the GNU gettext system see https://en.wikipedia.org/wiki/Gettext .
It really helps to know what the creators of gettext have meant things to be like.

You can find detailed information in  https://www.gnu.org/software/gettext/manual/index.html

## Install related tools

Current (as of 2016-03) Windows binaries of the general GNU gettext utilities can be obtained from
https://mlocati.github.io/gettext-iconv-windows/
Running the downloaded exe installs the binaries in a subdirectory of `C:\Program Files` with the option of adding that subdirectory to your `%PATH%` environment variable. Do use that option.


## Prepare Source Code

### Enclose all strings that need translation in `gettext("english version of translatable string")`.

The argument of gettext must always be one (1) single, uninterrupted literal string.

This rule imples:

* Never ever use line continuation inside the argument of gettext. Use variables and separate gettext lines instead.
* Never ever use variables inside the argument of gettext. If you need variables, do it this way:

```
StringFormat(gettext("My name is %s and I live in %s."), $sName, $sCity)
```
* Never ever use string concatenation inside the argument of gettext.
If you are composing long messages, do it like this:

```
MsgBox(..., ..., gettext("Translatable Part 1") & @CRLF & gettext("Translatable Part 2"))
```
* Always use double quotes to delimit the string literal of the gettext argument. Strings in single quotes will be silently discarded by xgettext.
* Do not attempt to use double double quotes from inside the gettext argument. The *only* method is this:

```
MsgBox(64, $apptitle, StringFormat(gettext("gettext_au3_language_select_ui() has returned %s%s%s."), '"', $gettext_au3_lang, '"'))
```
* Single quotes *inside* the double quoted gettext argument are o.k. though, e.g.

```
MsgBox(64, $apptitle, StringFormat(gettext("gettext_au3_language_select_ui() has returned '%s'."), $gettext_au3_lang))
```

To cater for situations when e.g. logging should be done in English, while the user is presented with userlang e.g. MsgBoxes, the global variable `$gettext_au3_sourceString` is appended to with the `source text & @CRLF` during each call of `gettext`.

### Include the Runtime Library

```
#include "gettext_au3_runtime_library.au3"
```

### Declare and Populate the Required Variables, `$gettext_au3_lang` must be Global

```
Local $apptitle = "Internationalized Test Application"
Global $gettext_au3_lang = gettext_au3_language_select_ui($apptitle, gettext_au3_language_list(), "en")
```

gettext_au3_language_select_ui does offer a 'Cancel' button. If the user choses to click it, gettext_au3_language_select_ui returns the special string "cancel" (without the quotes). This design assumes that there will never be a language code named "cancel".

## Generate the Template File (.pot)


The programmer needs to repeat this step after each modification to the source code that might have changed translatable strings, so he'll want to include this step into your makefile, generate.au3 or whatever build system you use.

The template file is one single file, common to all user languages, containing all the strings that need to be translated

```
xgettext -L C --from-code=UTF-8 --strict --omit-header --no-wrap -o messages.pot example-user-program.au3
```

Add source code files as needed. You need to specify every single .au3 file that might contain calls to gettext.

The sample xgettext_set.bat file included in this collection includes a call that does work for the example app.

### Copy Template file to Language-Specific PO file 

This needs to be done only once per language, usually after the first generation of `messages.pot`.

E.g. for language `de`:
```
copy messages.pot de.po
```

Very important: do edit the `Content-Type` header in the language-specific .po file to read `UTF-8` instead of `CHARSET`.

### Merge Updated Messages and Existing Translations

This needs to be done whenever a modified `messages.pot` was created, i.e. as a second step after each change of translatable strings in the source code.

```
msgmerge --no-wrap -U de.po messages.pot
```

The sample xgettext_set.bat file included in this collection includes calls for English and German that do work for the example app.

## Create the Translations

For each language, send the .po file to the translator, asking her to fill the `msgstr` lines.

Any UTF-clean and line-break preserving text editor, e.g. Notepad++, can do the job, but there are tools available that make this process easier. My personal favourite is PoEdit (https://poedit.net/).

Segments that have been changed since the last translation are marked with "fuzzy" by `msgmerge`.

## Generate the au3 source table

After the translations are back from the translators, run `gettext_au3_msgfmt.au3` (or its compiled version `gettext_au3_msgfmt.exe`), generating `gettext_au3_gettext.au3` from all `??.po` files in the current directory.
The output file name is hard coded in `msgfmt.au3` to ensure consistency with `gettext_au3_runtime_library.au3`, which includes this file.

## Compile a New Version of Your Application With Updated Translations

# Source Code Files

## `gettext_au3_runtime_library.au3`

Runtime library, to be included in the internationalized app.

Most importantly, `Func gettext($sSourceText)`, returning the translation of the parameter's value into language `$gettext_au3_lang`, is implemented here.

## `gettext_au3_msgfmt.au3`

Generator main program. Reads all `??.po` files (*language codes must have exactly two characters*) in the current directory and creates `gettext_au3_gettext.au3`.

## `gettext_au3_language_codes.au3`

List of some language codes, to be included in `gettext_au3_msgfmt.au3`, essentially `Func gettext_au3_language_name($language_code)` returning the name of the language of code par1 (two character string), in the language, e.g. "es" -> "Español".


## `gettext_au3_gettext.au3`

Generated code, to be included by the runtime. Contains these functions:

### `gettext_au3_runtime(Const $sSourceText, Const $lang)`

Returns the translation of par1 in the language par2.

### `gettext_au3_language_list()`

Returns a string containing the list of all languages that have translations in gettext_au3_runtime(.,.). Format e.g.:
"en,English|de,Deutsch"
The order of the languages usually is the alphabetical order of the language code (expect, but don't rely on this).
