---
created: 2024-05-28T00:30:58 (UTC +02:00)
tags: []
source: moz-extension://114c95b5-ba53-43e3-afa9-6de16943bae2/articles/how-to-get-emojis-working-on-linux/
author: Tanya Laskin
---

# How to Get Emojis Working on Linux

> ## Excerpt
> Some Linux distributions don't have proper fonts to display nice color emojis. From this article you'll learn how to fix this for your Linux.

---
January 18, 2023

 [![Share on Facebook icon](moz-extension://114c95b5-ba53-43e3-afa9-6de16943bae2/articles/img/sn/facebook.svg "Share this article on Facebook")](https://www.facebook.com/sharer/sharer.php?u=https://virola.io/articles/how-to-get-emojis-working-on-linux)[![Share on LinkedIn icon](moz-extension://114c95b5-ba53-43e3-afa9-6de16943bae2/articles/img/sn/linkedin.svg "Share this article on LinkedIn") ](https://www.linkedin.com/shareArticle?mini=true&url=https://virola.io/articles/how-to-get-emojis-working-on-linux)[![Share on Twitter icon](moz-extension://114c95b5-ba53-43e3-afa9-6de16943bae2/articles/img/sn/twitter.svg "Share this article on Twitter")](https://twitter.com/intent/tweet?url=https://virola.io/articles/how-to-get-emojis-working-on-linux&text=How%20to%20get%20emojis%20working%20on%20Linux)

___

The way emojis are displayed in Linux depends on the fonts installed in this operating system. If the right fonts are missing, or have no emojis included into them, or just have wrong configuration, emojis can look distorted, be mixed up or even not visible at all. This often happens in Linux Arch and CentOS but quite possible in other Linux distros as well.

To **install a font on Linux** you need to:

1.  Copy the font files to the specific directory
2.  Create a special configuration file

In this article, we will show you how to add a font and create a configuration file for **fontconfig library** that is widely used on different Linux operating systems to configure and customize font access.

## What font to choose to display emojis on Linux?

First, you need to choose a right font. You want to fix emojis appearance, thus you have to choose the font that includes emojis, preferably color ones. **We recommend using Noto Color Emojis font** created by Google. This font is free to use.

Linux uses fonts in **TrueType (TTF) format**.

You can download the font file from its [official GitHub repository](https://github.com/googlefonts/noto-emoji). The direct download link is the following: [https://github.com/googlefonts/noto-emoji/raw/main/fonts/NotoColorEmoji.ttf](https://github.com/googlefonts/noto-emoji/raw/main/fonts/NotoColorEmoji.ttf)

## How to install a font with emojis on Linux?

1.  After downloading the font TTF-file you need to **copy it to a special location**:
    
    -   `~/.local/share/fonts` – if you want to install it only _for the current Linux user_
    -   `/usr/share/fonts` – if you want to install it _globally for the whole system_
    
    If mentioned directories are missing, you need to create them. **We recommend installing the font globally**, but make sure that you have administrator rights to work with system directories.
2.  Now you need to **create a special configuration file** that will instruct fontconfig library to use the new font instead of the default system one if it doesn't contain emojis.
    -   If you installed the font just _for the current user_, create the configuration file `~/.config/fontconfig/fonts.conf`
    -   If you installed the font _globally_, create the configuration file `/etc/fonts/local.conf`If the file already exists (this is possible if you've previously configured fonts on your Linux), you'll need to edit it.
3.  **Add the following text** to your `fonts.conf` file:
    
    ```
        <code>
    &lt;?xml version="1.0"?&gt;
    &lt;!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd"&gt;
    &lt;fontconfig&gt;
        &lt;alias&gt;
            &lt;family&gt;sans-serif&lt;/family&gt;
            &lt;prefer&gt;
                &lt;family&gt;Noto Color Emoji&lt;/family&gt;
            &lt;/prefer&gt;
        &lt;/alias&gt;
    
        &lt;alias&gt;
            &lt;family&gt;serif&lt;/family&gt;
            &lt;prefer&gt;
                &lt;family&gt;Noto Color Emoji&lt;/family&gt;
            &lt;/prefer&gt;
        &lt;/alias&gt;
    
        &lt;alias&gt;
            &lt;family&gt;monospace&lt;/family&gt;
            &lt;prefer&gt;
                &lt;family&gt;Noto Color Emoji&lt;/family&gt;
            &lt;/prefer&gt;
        &lt;/alias&gt;
    &lt;/fontconfig&gt;
        </code>
    ```
    
4.  **Clear the system font cache** using the `fc-cache` command
5.  **Restart the application** that has issues with emojis for the changes to take effect. Sometimes **system restart** may be required.

## Using emojis in Virola Messenger on Linux

In [Virola Messenger](moz-extension://114c95b5-ba53-43e3-afa9-6de16943bae2/get-virola/self-hosted-server), we have our own set of emojis that can be added to chat messages via the emoji menu. These emojis are displayed in chat texts. However, on chat tabs and tray notifications emojis from the system font are currently used. Therefore, if you don't have a system font that includes nice color emojis, you may see unattractive monochrome emojis there, or even see nothing.

This guide was initially created to fix the issue with absent / incorrect emojis on Linux in Virola Messenger. But in the [latest versions of Virola client](moz-extension://114c95b5-ba53-43e3-afa9-6de16943bae2/get-virola/client) we included the needed font to the installation package, so the fix may be not required now.

However, **this solution may be used to fix emojis in other Linux applications**.
