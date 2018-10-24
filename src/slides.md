---
title: Writing a Screencast Video Editor in Haskell
author: Oskar Wickström
date: October 2018
theme: Boadilla
classoption: dvipsnames
---

## About Me

- Live in Sweden, Malmö
- Work for [Symbiont](https://symbiont.io/)
- Blog at [wickstrom.tech](https://wickstrom.tech)
- [Haskell at Work](https://haskell-at-work.com) screencasts
- Maintain some open source projects

# Background{background=images/evolution.png .contain}

## Haskell at Work

- "Haskell at Work" screencasts
    - Centered around code editor
    - Fast-paced
    - No webcam overlay or effects
- Workflow:
    - Write a script
    - Record video separately
    - Record audio separately
    - Cut and join all "scenes"

## Video Editors

- Free video editors: Kdenlive, OpenShot
    - No good normalization and gate audio effects
    - Unsuited for my workflow
    - Unstable (and I don't want to hack C++)
- Commerical video editors: Premiere Pro, Final Cut Pro
    - Proprietary
    - Expensive
    - Unsuited for my workflow

# The Great Yak Shave{background=images/yak.jpg .dark}

## Building a Video Editor

- I decided to build a screencast video editor
    - Minimal
    - Suited to my workflow
- I decided write it in Haskell
    - Didn't want to write an Electron app
    - Settled on GTK+
- Dog-fooding some of my own libraries

# Komposition{background=#00008B .dark}

## Komposition Overview

<table>
  <tr>
    <td>
- Modal
- Hierarchical timeline
    - Sequences
    - Parallels
    - Clips and gaps
- Automatic scene classification
- Automatic sentence classification
- Keyboard-driven editing workflow
- Cross-platform
    </td>
    <td width="50%">
![Komposition](images/komposition.png)
    </td>
  </tr>
</table>

## Documentation

- [owickstrom.github.io/komposition/](https://owickstrom.github.io/komposition/)
    - User guide
    - Tutorial screencast

![Komposition documentation](images/documentation.png){width=60%}

## Hiearchical Timeline

- **Sequences** contain parallels that are played in sequence
- **Parallels** contain video and audio tracks that are played in parallel
- **Gaps** are rendered with still frames or silence
- Same for shorter tracks within a parallel

## Keyboard-Driven Editing

- Vim-like bindings
- Corresponding menu items
- Some mouse support
- Help dialog showing current mode's key bindings

# Demo{background-video=images/demo.gif background-video-loop=true .dark}

# Implementation{background=images/cogs.jpg .dark}

## Striving for Purely Functional

- Pure functions and data structures in core domain
    - Timeline
    - Focus
    - Commands
    - Event handling
    - Key bindings
- Impure parts:
    - Audio and video import
    - Scene/sentence classification
    - Preview frame rendering
    - Main application control flow

## GTK+

- Haskell bindings from `gi-gtk`
- Regular GTK+ was too painful
    - Imperative
    - Callback-oriented
    - Everything in IO, no explicit model
- Started `gi-gtk-declarative`
    - Declarative using data structures
    - VDOM-like diffing
    - Events are functions and values
    - Custom widgets
- Imperative `gi-gtk` where needed

## Type-Indexed State Machines

- Using `motor` for describing state machines with types:

    ```haskell
    start
      :: Name n
      -> KeyMaps
      -> Actions m '[ n !+ State m WelcomeScreenMode] r ()
    ```

    ```haskell
    returnToTimeline
      :: ReturnsToTimeline mode
      => Name n
      -> TimelineModel
      -> Actions m '[ n := State m mode !--> State m TimelineMode] r ()
    ```
- Most complicated aspect of the codebase
- Maybe worth it
- Currently being rewritten

## Automatic Scene Classification

- Creates a producer of frames

    ```haskell
    readVideoFile :: MonadIO m => FilePath -> Producer (Timed Frame) m ()
    ```
- Custom algorithm for classification

    ```haskell
    classifyMovement
        :: Monad m
        => Time -- ^ Minimum segment duration
        -> Producer (Timed RGB8Frame) m ()
        -> Producer (Classified (Timed RGB8Frame)) m ()

    classifyMovingScenes ::
         Monad m
      => Duration -- ^ Full length of video
      -> Producer (Classified (Timed RGB8Frame)) m ()
      -> Producer ProgressUpdate m [TimeSpan]
    ```

## Automatic Sentence Classification

- Automatic sentence classification of audio
- Currently using `sox`
    - Normalization
    - Noise gate
    - Auto-splitting by silence
- Hard to find good defaults (currently hard-coded)
- Creates segment audio files on disk (can't extract timespans)

## Rendering

- Flattening timeline
    - Conversion from hierarchical timeline to flat IR
    - Pads gaps and empty parts with still frames
- Data types for FFmpeg CLI syntax
    - Common flags
    - Filter graph

## Preview

- Proxy media for performance
- Same FFmpeg backend as when rendering
    - Output is a streaming HTTP server
    - Not ideal, would like to use a named pipe or domain socket
- GStreamer widget
    - Consumes the HTTP stream
    - Embedded in the GTK+ user interface
- Unreliable
- Currently doesn't work on individual clips and gaps

# Testing{background=images/testing.jpg .dark}

## Color-Tinting Video Classifier

- Tints the original video with red/green based on classification
- Easier to test classifier on real recordings

![Komposition](images/color-tinting.gif)

## Property-Based Testing

- Timeline commands and movement
    - Generates sequences of commands
    - Applies all commands
    - Resulting focus should always be valid
- Video scene classification
    - Generates known test scenes
    - Translates to real pixel buffers
    - Runs classifier, compares to known test scenes
- Flattening of hierchical timeline
- Symmetry of FFmpeg format printers and parsers

## Example-Based Testing

- Commands
- Navigation
- FFmpeg syntax printing

# Used Packages

## haskell-gi

- Bindings for GTK+, GStreamer, and more
    - gi-gobject
    - gi-glib
    - gi-gst
    - gi-gtk
    - gi-gdk
    - gi-gdkpixbuf
    - gi-pango
- Extended with gi-gtk-declarative

## massiv & massiv-io

- Used in video classifier
- Parallel comparison of pixel arrays
- Conversion from JuicyPixels frames to massiv arrays
- Lower-resolution proxy media helps with performance

## Pipes

- Streaming frame reader and writer around `ffmpeg-light`
- IO operations with streaming progress notifications

    ```haskell
    importVideoFileAutoSplit
      :: (MonadIO m, MonadSafe m)
      => VideoSettings
      -> FilePath
      -> FilePath
      -> Producer ProgressUpdate m [VideoAsset]
    ```
- `pipes-safe` for handling resources
- `pipes-parse` for `StateT`-based transformations

## Others

- protolude
- lens
- typed-process

# Summary

## Retrospective

- The best parts:
    - Haskell and GHC
    - Keeping core domain pure
    - Testing with Hedgehog
    - Making a useful tool
- The problematic parts:
    - Video and audio codecs, containers, streaming
    - Executing external programs
    - GTK+ in Haskell
    - Packaging and dependencies

## Next Steps

- Features
    - More commands (yank, paste, join, ...)
    - Preview any timeline part
    - Adjust clips
    - Better OS integration and GUI
- Improvements
    - Technical debt, refactoring
    - Content-addressed project files (reuse, avoiding collision)
    - Optimized FFmpeg rendering
    - Optimized diffing (gi-gtk-declarative)
- Packaging (Debian, macOS, Windows, nixpkgs)

## Thank You!

- Komposition: [owickstrom.github.io/komposition/](https://owickstrom.github.io/komposition/)
- Slides: [owickstrom.github.io/writing-a-screencast-video-editor-in-haskell/](https://owickstrom.github.io/writing-a-screencast-video-editor-in-haskell/)
- Code examples and slides source code: [github.com/owickstrom/writing-a-screencast-video-editor-in-haskell/](https://github.com/owickstrom/writing-a-screencast-video-editor-in-haskell/)
- Image credits:
    - [Yak by travelwayoflife - Flickr, CC BY-SA 2.0](https://commons.wikimedia.org/w/index.php?curid=22106967)
    - [Human evolution scheme by M. Garde - Self work (Original by: José-Manuel Benitos), CC BY-SA 3.0](https://commons.wikimedia.org/w/index.php?curid=2165296)
    - [Old Cogs by Emmanuel Huybrechts from Laval, Canada (Old Cogs) CC BY 2.0, via Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Old_Cogs_(5084228263).jpg)
    - [Save the skateboard!](https://www.reddit.com/r/BetterEveryLoop/comments/6gsbs3/save_the_skateboard/)
