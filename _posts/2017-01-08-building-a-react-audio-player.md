---
layout: post
title: Building a React Audio Player
---

I'm going to try to something different for this post. I'm going to write and blog about building a react audio player at the same time. You can check the result on [github](https://github.com/maestroh/streamer).

## Result
![Player](/img/player.png)

My goal is to build a player that looks similar to the one above. It will only need a couple of features: a play/pause button and a progress bar. I was able to test the rest API that sends a stream of music by using the default audio player. It worked great but the player looks a bit clunky, can't be styled and has features I don't want.

## Real Quick
Before I get started with the audio bits, I've got to write the component itself. I've also included the audio component. I know I'll need a reference the audio element to playback and control the audio. The component accepts a prop called `audio` containing the url to the audio stream.

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
To get started, i'm going to work on the simplest part: the play & pause buttons. First step is to get a couple of icons. I going to use the `ion-play` and `ion-pause` icons from [ionicons](http://http://ionicons.com/). Before I wire up the audio element, I think I'll just toggle the icons.

Alright, when this button gets clicked, it should toggle back and forth between the two icons. I'll need some state and a function to toggle that state on the click. The snippet below has a constructor initializing the `play` state.

```
constructor(props) {
    super(props);
    this.state = { play: false };
  }
```
Then I'll toggle the state with a function.

```
  play = () => {
    this.setState({play: !this.state.play});
  }
```
Finally, I'll add a `div` component and select the correct icon class based on the state.

```
<div onClick={this.play} className={!this.state.play ? "icon ion-play" : "icon ion-pause"} />
```
