# calibre-symlinker

Create a mirrored Calibre e-Books library without duplicating it

## Why?

This project started due to Calibre limitation to set custom library format since
understandably it'll mess the internal database.

While this is working if you're only using Calibre both as your library manager
and viewer, it would be different matter if you want to share the library to
other apps—especially when you're also using Jellyfin as media manager.

## How?

This application will create a mirror library from your current Calibre Library
path. By utilizing sidecar `.opf` files provided when user imported the book to
Calibre, this app will recursively reads its content then decide how to format
the library based on default configuration or [user-defined basic template](#template).

## Limitation

As you can read from [#How?](#how), the process on mirroring your library is
extremely limited to books with `.opf` files available inside on your book
folder. By default, `.opf` file will be generated and modified when you imported
a book to Calibre. Unless you've restored the library using backup feature or
disabled file generation, you does not need any modification to your library.

You can generate new `.opf` files if its missing from your book folder by
following these steps:

1. Open your Terminal/Command Prompt.
2. Create a new folder destination by typing following command:

   ```bash
   mkdir calibreOpf
   cd calibreOpf
   ```

3. Depending `calibredb` is available on your `PATH`, copy following line, paste,
   and execute it:

   ```bash
   calibredb export --all --progress --formats "" --template "{authors:sublist(0,1, & )}/{title:.96:swap_around_articles(,)} ({id})/metadata" --dont-save-cover --dont-save-extra-files
   ```

   * If `calibredb` was not found:
     * Install from your package manager, if any, on Linux.
     * On Windows and assuming install path of Calibre set to default, change
       `calibredb` to `"C:\Program Files\Calibre2\calibredb.exe"`.
   * If the library was not found, or in a network (why? it aint the tool tho?):

     Consider enable sharing over network on Calibre GUI, and add additional
     option `--library-path http://localhost:8080/#mylibrary`. See more about
     the option in [man page][ubuntumanpage]

4. Open your file manager by executing `explorer .` on Windows or `nautilus .`
   on Linux with GNOME DE.... or whatever you use in other DE... or you can
   use trusty `cp` command if you know what are you doing. 👀

5. Select all folders inside and copy. Paste it to your Calibre Library folder.
   If file manager found a duplicate `.opf` file, skip all to avoid any issues.
   Don't close the file manager yet, you'll need it later.

6. Finally, on your terminal, change directory to your Calibre Library folder.

7. Check any orphan `.opf` files inside the library by utilizing `tree /F` on
   Windows, or simply `tree`. If you found any, move the file to its correct
   book folder by matching its ID. It's recommended to set your terminal and
   file manager windows side-by-side so you can compare it.

## Template

This program uses different variables and template approach than Calibre. By
default, the tool uses `{az_series_or_title}/{series}/{title}` for its mirror
library, and yes, the app will automatically sanitize the path.

As user, you can override default behaviour by using following built-in
variables:

### Available variables

#### Authors

* `a`, `author`: Author. Only grab unsorted, first name (`Anonymous Writer`)
* `as`, `authors`: Authors. All unsorted author names, delimited by `&`
  (`Anonymous Writer & Mophy Ezmee`)
* `aza`, `az_author`: Author, only grab first letter from unsorted name (`A`)
* `sa`, `sort_author`: Author, sorted name. Only grab first name (`Writer, Anonymous`)
* `sas` `sort_authors`: Authors. All sorted author names, delimited by `&`
  (`Writer, Anonymous & Ezmee, Mophy`)
* `azsa`, `az_sort_author`: Author, only grab first letter from sorted name (`W`)

#### Identifiers

* `calibreid`: Calibre ID
* `uuid`: UUID
* `isbn`: ISBN
* `googleid`: Google
* `amazonid`, `asin`: Amazon
* `goodreadsid`: Goodreads

#### Series and title

* `s`, `series`: Book series
* `azs`, `az_series`: Book series. Only grab first letter
* `st`, `series_or_title`: Book series or title. Prioritize series name if exist.
* `azst`, `az_series_or_title`: Book series or title. Only grab first letter
* `t`, `title`: Book title
* `azt`, `az_title`: Book title. Only grab first letter

#### Volume cataloging

* `v`, `vol`, `volume`: Volume, in arabic numeral (1, 2, 3, ...)
* `rv`, `rvol`, `roman_volume`: Volume, in Roman numeral (I, II, III, ...).
  Volume with additional subvolume (e.g. `1.50`) will be reverted to arabic
  numeral instead.

#### Date

> [!NOTE]
> If you want to use added date, append `_add` at the end of variable name
> instead.

* `y`, `year`: Release year
* `m`, `month`: Release month, in number
* `mm`, `mmonth`: Release month, in system-configured language. (January)
* `azmm`, `az_mmonth`: Release month, in system-configured language. Only grab
  first letter,
* `d`, `day`: Release day, in number
* `wd`, `weekday`: Release weekday name, in system-configured language. (Monday)

#### Misc

* `lang`, `language`: ISO3B language code
* `pub`, `publisher`: Publisher

### String manipulation

soon. 👍

> [!IMPORTANT]
> While using A-Z (grab first letter) variables, the app will respect article
> words from a language unless the variable was prepended with `dia_` (stand for
> *don't ignore articles*), for example, setting up `dia_azst` for "Grab first
> letter from series or title, don't ignore article words" on `"The Ones Within"`
> book series will return `T` rather `O`.
>
> By default, the app will ignore English articles. To override this behaviour,
> read more `--ignore-articles` option.

<!-- Footnotes and References -->
[ubuntumanpage]: https://manpages.ubuntu.com/manpages/noble/en/man1/calibredb.1.html#global%20options
