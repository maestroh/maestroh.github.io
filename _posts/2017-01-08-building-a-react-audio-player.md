---
layout: post
title: Building a React Audio Player
---

I'm going to try to something different for this post. I'm going to write and blog about building a react audio player at the same time. You can check the result on [github](https://github.com/maestroh/streamer).

## Result
![Player](/img/player.png)

My goal is to build a player that looks similar to the one above. It will only need a couple of features: a play/pause button and a progress bar. I was able to test the rest API that sends a stream of music by using the default audio player. It worked great but the player looks a bit clunky, can't be styled and has features I don't want.

## Real Quick
Before I get started with the audio bits, I've got to write the component itself. I've also included the audio component. I know I'll need a reference the audio component to playback and control the audio.

```
import React from 'react';

export default class Player extends React.Component {
    render() {
        return  <audio src={this.props.audio}
                    autoPlay
                />
    }
}
```

## Play/Pause Button
To get started, i'm going to work on the simplest part, theI've started out with the play & pause buttons. 
