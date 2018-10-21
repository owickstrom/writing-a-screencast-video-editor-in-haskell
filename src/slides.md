---
title: Writing a Screencast Video Editor in Haskell
author: Oskar Wickström
date: October 2018
theme: Boadilla
classoption: dvipsnames
---

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
    - Settled on GTK+ (`gi-gtk` package)
- Dog-fooding some of my own libraries

# Komposition{background=images/cinema.jpg .dark}

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
    </td>
    <td width="50%">
![Komposition](images/komposition.png)
    </td>
  </tr>
</table>

## Hiearchical Timeline

- _Sequences_ contain _parallels_ that are played in sequence
- _Parallels_ contain video and audio tracks that are played in parallel
- Gaps and shorter tracks are _filled_ with still frames or silence

## Keyboard-Driven Editing

- Vim-like bindings
- Data structures
    - Events
    - Keymaps
- Help dialog
    - Shows all key bindings

# Implementation

## Pure core domain

- Timeline
- Focus
- Commands

## Property-Based Testing

- Timeline commands and movement
    - Generates sequences of commands
    - Applies all commands
    - Resulting focus should always be valid
- Video scene classification
    - Generates known test scenes
    - Translates to real pixel buffers
    - Runs classifier, compares to known test scenes
- Symmetry of FFmpeg format printers and parsers

## GTK+

- Regular GTK+ was too painful
- `gi-gtk-declarative`
- Custom widgets
- Imperative `gi-gtk` where needed

## Automatic Scene Classification

- Automatic scene classification of video
- Using `ffmpeg-light` to create a `Producer` of frames
- Custom algorithm for classification
- `massiv` and `massiv-io` for parallel pixel comparison
- Property-based testing with Hedgehog

## Automatic Sentence Classification

- Automatic sentence classification of audio
- Currently using `sox`
- Normalization
- Noise gate
- Auto-splitting by silence

## Rendering

- Flattening timeline
- Pads gaps and empty parts with still frames
- Data types for FFmpeg CLI syntax
    - Common flags
    - Filter graph

## Preview

- Proxy media for performance
- Same FFmpeg backend as when rendering
- Streaming to GStreamer widget

# Used Packages

## haskell-gi

- Bindings for GTK+, GStreamer, and more
- ...

## gi-gtk-declarative

- Declarative GTK+
- ...

## ffmpeg-light

- Streaming frame reader
- ...

## massiv & massiv-io

- Parallel comparison of pixel arrays
- ...

## Pipes

- Streaming frame reader around ffmpeg-light
- IO operations with streaming progress notifications

## Motor

- Finite-state machines with type-safe transitions and effects

## Lens

- lens
- ...

# Summary

## Thank You!

- Slides: [owickstrom.github.io/declarative-gtk-programming-in-haskell/](https://owickstrom.github.io/declarative-gtk-programming-in-haskell/)
- Code examples and slides source code: [github.com/owickstrom/declarative-gtk-programming-in-haskell/](https://github.com/owickstrom/declarative-gtk-programming-in-haskell/)
- Image credits:
    - Yak: [by travelwayoflife - Flickr, CC BY-SA 2.0](https://commons.wikimedia.org/w/index.php?curid=22106967)
    - Human evolution scheme: [by M. Garde - Self work (Original by: José-Manuel Benitos), CC BY-SA 3.0](https://commons.wikimedia.org/w/index.php?curid=2165296)