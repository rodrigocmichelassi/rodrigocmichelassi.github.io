---
title: "[GNOME] Displaying bitrate information on audio files"
date: 2025-05-14 21:00:00 +/-0300
categories: [Open Source Software Development, GNOME contributions]
description: "In this post, we will take a closer look at the GNOME Files issue 3856, in which we want to show the bitrate information for audio files when accessing its properties"
tags: [linux, gnome, setup, contribution, floss, open source, mac0470]
---

In the last post, we setted up an environment to start contributing to the GNOME project. With that done, we can now start studying and taking a look inside the project. GNOME has some apps that are well documented by its mantainers, and for new contributors like ourselves, we can look for issues carrying the newcomers tag. We chose to contribute to the Nautilus/Files project.

### Choosing the first issue

Looking for an issue tagged as newcomers on the files project, we came across the [issue 3856](https://gitlab.gnome.org/GNOME/nautilus/-/issues/3856#note_2438004), where a user reported that, in android, audio files, specially the ones with `.FLAC` extension, usually display bit rate information on the properties tab, but that was not happening on GNOME for desktop, for some reason.

At first, we chose this issue because there was a suggestion from one of the mantainers, where he stated that the issue should be easily resolved using the `gst_player_audio_info_get_bitrate` method from the GStreamer library. I felt really confused at first not knowing where to use this, and if it would be really doable with that, so I started looking for the code that defines the properties window and found the `totem-properties-view.c` file, which was inside the audio-video-properties directory, which truly suited our problem. 

![Bitrate info not showing]({{ '/assets/img/2025-05-14-first-contribution-to-gnome/audio-without-bitrate.png' | relative_url }})
_Audio properties without bitrate information_

Taking a closer look at this file, I actually saw that, apparently, the code was already getting the bitrate information using the `gst_discoverer_audio_info_get_bitrate` method, from the same library. Being confused with everything, I posted a comment on the issue, questioning the use of this, and another mantainer said that, indeed, the bitrate info was being obtained, but the return value was 0. With that, the bitrate information was ignored on the `set_bitrate` function:

```c
static void
set_bitrate (TotemPropertiesView *props,
             guint                bitrate,
             const char          *title)
{
    char *string;

    if (!bitrate)
    {
        return;
    }
    string = g_strdup_printf (_("%d kbps"), bitrate / 1000);
    append_item (props, title, string);
    g_free (string);
}
```

The mantainer also removed this issue from the newcomers tag, which felt a little scary at first, cause this should be a little more difficult than expected to solve.

### Testing and first attempt

Knowing that we were not being able to get the bit rate information by the use of this library, we actually did not give up completely on it. By studying the code, I saw that this library was actually used to get lots of metadata from audio files, and those metadata were actually on display, so my first thought was that the bit rate information might not be available as a metadata, or the library had applied something wrong. By reading a little more about `.FLAC` files, I found that [it does not store bit rate information](https://datatracker.ietf.org/doc/rfc9639/) as a metadata, as we first suspected.

If we do not have bit rate information on the file, probably we should just let it go, right? Wrong! It is still possible to obtain this value in some other ways. We could manually calculate this number on the `update_audio` function, by adding some new parameters. Our first try was to calculate the bit rate as _bit-rate = sample-size x number-of-samples x number-of-channels_, a general theoretical result to obtain the bitrate. With that done on the code, by using the own file metadata, we noticed that the bit rate value was actually being displayed when testing.

In spite of that working, there was still a problem. This equation is a fully theoretical result, and does not truly match reality, so the bit rate value we were calculating did not reflect the actual bit rate for `.FLAC` files. That happens because that kind of file deals with compression, that is not taken as part of the calculation by using that equation. That way, the result would overrate the actual bit rate. On the other hand, we could still calculate it by using the _bit-rate = filesize (bytes) x 8 / duration (s)_ equation.

### Final attempt and result

With all that was discussed in mind, our goal now was just to get a way to apply the last equation in order to calculate the bit rate value. For that, we had to roll over to the `discovered_cb` function, that is used to call the `update_audio` function. In that function, we were able to access the `GstDiscovererInfo` parameter and get the audio file duration, something we could not really do on the `update_audio` function. As a matter of fact, the duration was already being calculated in this current function, so we could only use that result. We were also able to access the file URI and get the file data inside the code.

```c
GFile *file = g_file_new_for_uri (gst_discoverer_info_get_uri (info));

if (audio_streams)
{
    update_audio (props, audio_streams->data, file, duration);
}
```

As it is possible to see, we were forced to add two new parameters on the update_audio function, in order to explicitly calcualte the bit rate value. After all that process, we were able to retrieve information from the audio file and get the file size:

```c
guint64 duration_seconds;
goffset filesize;
GFileInfo *fileinfo;

fileinfo = g_file_query_info (file, G_FILE_ATTRIBUTE_STANDARD_SIZE, G_FILE_QUERY_INFO_NONE, NULL, NULL);
if (fileinfo) 
{
    filesize = g_file_info_get_size (fileinfo);
    duration_seconds = duration / GST_SECOND;

    g_object_unref (fileinfo);
}
else 
{
    duration_seconds = 0;
}
g_object_unref (file);
```

Now, we can finally calculate the bit rate information:

```c
if (filesize)
{
    bitrate = filesize * 8 / duration_seconds;
}
else 
{
    bitrate = gst_discoverer_audio_info_get_bitrate (info);
}

set_bitrate (props, bitrate, _("Audio Bit Rate"));
```

Now, it is time to finally test our function and check if it all worked as supposed. First of all, we got our sample `.FLAC` file and ran on an onine bit rate calculator, that returned the 943kbps value. Now, we can compare our calculated value obtained from this code implementation, that calculates an approximated bit rate value:

![Bitrate info]({{ '/assets/img/2025-05-14-first-contribution-to-gnome/audio-with-bitrate.png' | relative_url }})
_Audio properties with bitrate information being displayed_

As it is possible to see, the bit rate information achieved was really close to the bit rate calculated by the online website. It is possible that the website implementation considers other parameters to calculate the bit rate information in a more precise way, but this still needs to be evaluated. We also tested for other files and the result was always this close.

For now, we have solved the main problem for this issue, only for audio files, as first reported, and we are waiting for the contributors to answer our final comment on the issue, in order to send our first MR to the GNOME project! Hopefully, I will be updating this page soon with an acceptance on this MR.

### Updates on this contribution

It took a lot of effort into fixing things to submit a MR, but at final it was ok. Got no complains on the way our code was implemented, but not everything is the way we wish it was. 

After not having any answers for days, we decided to submit the contribution, but it did not get accepted. To be honest, it actually feels like the mantainers are correct in this, they said that it is better that we do not manually calculate the bitrate information to be displayed on the properties window, since the other properties are given from the GStreamer library. As a matter of fact, I still feel like we had a nice part contributing to this, since the mantainers were able to create an issue on the GStreamer gitlab.

![Interaction with mantainers]({{ '/assets/img/2025-05-14-first-contribution-to-gnome/interaction.png' | relative_url }})
_Final interaction with the GNOME mantainers_

Now, I am not sure we can get the bitrate information on the GStreamer lib the same way we did to calculate it manually here for GNOME Nautilus, but I sure will take a look and check if it is possible to contribute to the GStreamer lib. Anyway, it was a nice path followed and I have learnt a lot in the process, and I am really looking forward to contribute more to GNOME.

> This post is used as a checkpoint on GNOME contributions, first started as an assignment on the [Open Source development class](https://uspdigital.usp.br/jupiterweb/obterDisciplina?sgldis=MAC0470&codcur=3122&codhab=5000), at the Institute of Mathematics and Statistics of the University of São Paulo, advised by professor [Paulo Meirelles](https://www.ime.usp.br/~paulormm/).
{: .prompt-info }