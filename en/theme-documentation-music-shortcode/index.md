# Theme Documentation - music Shortcode


The `music` shortcode embeds a responsive music player based on [APlayer](https://github.com/MoePlayer/APlayer) and [MetingJS](https://github.com/metowolf/MetingJS).

<!--more-->

There are three ways to use it the `music` shortcode.

## 1 Custom Music URL {#custom-music-url}

{{< version 0.2.10 >}} The complete usage of [local resource references](../theme-documentation-content#contents-organization) is supported.

The `music` shortcode has the following named parameters by custom music URL:

- **server** _[required]_

  URL of the custom music.

- **name** _[optional]_

  Name of the custom music.

- **artist** _[optional]_

  Artist of the custom music.

- **cover** _[required]_

  URL of the custom music cover.

Example `music` input by custom music URL:

```markdown
{{</* music url="/music/Wavelength.mp3" name=Wavelength artist=oldmanyoung cover="/images/Wavelength.jpg" */>}}
```

The rendered output looks like this:

{{< music url="/music/Wavelength.mp3" name=Wavelength artist=oldmanyoung cover="/images/Wavelength.jpg" >}}

## 2 Music Platform URL Automatic Identification {#automatic-identification}

The `music` shortcode has one named parameter by music platform URL automatic identification:

- **auto** _[required]_ (**first** positional parameter)

  URL of the music platform URL for automatic identification,
  which supports `netease`, `tencent` and `xiami` music platform.

Example `music` input by music platform URL automatic identification:

```markdown
{{</* music auto="https://music.163.com/#/playlist?id=60198" */>}}
Or
{{</* music "https://music.163.com/#/playlist?id=60198" */>}}
```

The rendered output looks like this:

{{< music auto="https://music.163.com/#/playlist?id=60198" >}}

## 3 Custom Server, Type and ID {#custom-server}

The `music` shortcode has the following named parameters by custom music platform:

- **server** _[required]_ (**first** positional parameter)

  [`netease`, `tencent`, `kugou`, `xiami`, `baidu`]

  Music platform.

- **type** _[required]_ (**second** positional parameter)

  [`song`, `playlist`, `album`, `search`, `artist`]

  Type of the music.

- **id** _[required]_ (**third** positional parameter)

  Song ID, or playlist ID, or album ID, or search keyword, or artist ID.

Example `music` input by custom music platform:

```markdown
{{</* music server="netease" type="song" id="1868553" */>}}
Or
{{</* music netease song 1868553 */>}}
```

The rendered output looks like this:

{{< music netease song 1868553 >}}

## 4 Other Parameters {#other-parameters}

The `music` shortcode has other named parameters applying to the above three ways:

- **theme** _[optional]_

  {{< version 0.2.0 changed >}} Main color of the music player, default value is `#448aff`.

- **fixed** _[optional]_

  Whether to enable fixed mode, default value is `false`.

- **mini** _[optional]_

  Whether to enable mini mode, default value is `false`.

- **autoplay** _[optional]_

  Whether to autoplay music, default value is `false`.

- **volume** _[optional]_

  Default volume when the player is first opened, which will be remembered in the browser, default value is `0.7`.

- **mutex** _[optional]_

  Whether to pause other players when this player starts playing, default value is `true`.

The `music` shortcode has the following named parameters only applying to the type of music list:

- **loop** _[optional]_

  [`all`, `one`, `none`]

  Loop mode of the music list, default value is `none`.

- **order** _[optional]_

  [`list`, `random`]

  Play order of the music list, default value is `list`.

- **list-folded** _[optional]_

  Whether the music list should be folded at first, default value is `false`.

- **list-max-height** _[optional]_

  Max height of the music list, default value is `340px`.

