---
layout: post
title: Building a React Audio Player
---

I'm going to try to something different for this post. I'm going to write and blog about building a react audio player at the same time. You can check the result on [github](https://github.com/maestroh/streamer).

## Result
![Player](/img/player.png)

My goal is to build a player that looks similar to the one above. It will only need a couple of features: a play/pause button and a progress bar. I was able to test the rest API that sends a stream of music by using the default audio player. It worked great but the player looks a bit clunky, can't be styled and has features I don't want.

## Real Quick
Before I get started with the audio bits, I've got to write the component itself. I've also included the audio component. I know I'll need a reference the audio element to playback and control the audio. The component accepts a prop called `audio` containing the URL to the audio stream.

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
To get started, i'm going to work on the simplest part: the play & pause buttons. First step is to get a couple of icons. I am going to use the `ion-play` and `ion-pause` icons from [ionicons](http://http://ionicons.com/). Before I wire up the audio element, I think I'll just toggle the icons.

Alright, when this button gets clicked, it should toggle back and forth between the two icons. I'll need some state and a function to toggle that sets the state on each click. The snippet below has a constructor initializing the `play` state.

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
Looks good so far. The last bit of functionality for the button is to wire it up to the audio element. The audio element API has a play and pause function. I'll have to create a reference to the audio element using the React component `ref` attribute. According to the [React docs](https://facebook.github.io/react/docs/refs-and-the-dom.html), the reference to the audio element can be stored in a variable on the class instead of the state of the component. I'll start there.

```
<audio src={this.props.audio}
        ref={(audio) => { this.audio = audio } }
        autoPlay
        />
```
I've just stored the audio element itself in a property on the class. Next step is to call the audio element's API to play and pause playback. The good news is that I already wrote a `play` function. I'll just have to change it up a bit to call the API.

```
play = () => {
    if (this.state.play) {
      this.setState({ play: false });
      this.audio.pause();
    } else {
      this.setState({ play: true });
      this.audio.play();
    }
  }
```

There's a tiny bug in the above code. When the new URL is passed in, the state of the button is in the play state which displays the play button instead of the pause button. The fix is to change the state to a play state when the props change.

```
componentWillReceiveProps() {
    this.setState({ play: true });
}
```
## Timeline

Now things get complicated. I've got to create a timeline with a handle. I want to drag the handle to a specific point in the audio. I also want to move the handle to any point in the timeline when I click on the timeline. I've got to create the UI before any functionality gets added. 

That's probably the easiest part. I can just add a couple of `div` tags . . .

```
<div id="timeline">
    <div id="handle" />
</div>
```

and style them to look like the timeline in the mockup above.

```
#timeline{
    width: 400px;
    height: 20px;
    border-radius: 15px;
    background: rgba(0,0,0,.3);
}

#handle{
	width: 18px;
	height: 18px;
	border-radius: 50%;
	margin-top: 1px;
	background: rgba(0, 0, 0,1);
}
```

Not exactly the same look, but close enough. I think my first task will be to move the handle after getting selected and moving the mouse. To do that, I'll need the position of the handle and the timeline. To get that, I'll need the references to the underlying elements.

```
<div id="timeline" ref={(timeline) => { this.timeline = timeline }}>
	<div id="handle" ref={(handle) => { this.handle = handle }} />
</div>
```

### Dragging the Handle
Awesome! Now I've got the elements. I can start doing some math. When the handle gets clicked, I've got to move the handle to where the mouse moves, but I can't move the handle outside of the timeline. I'll need some data: the timeline width and the position of the handle based on the left boundary of the timeline and the X-position of the mouse.

#### MouseMove Handler
```
mouseMove = (e) => {
  // Width of the timeline
  var timelineWidth = this.timeline.offsetWidth - this.handle.offsetWidth;

  // Left position of the handle
  var handleLeft = e.pageX - this.timeline.offsetLeft;
}
```
Based on that data, the handle can now follow mouse. The only catch is that the mouse position may go outside of the timeline. I have to set limits around changing the handle position.
```
if (handleLeft >= 0 && handleLeft <= timelineWidth) {
  this.handle.style.marginLeft = handleLeft + "px";
}
```
The final task is to set the handle position if the mouse strays outside of the bounds of the timeline.
```
if (handleLeft < 0) {
  this.handle.style.marginLeft = "0px";
}
if (handleLeft > timelineWidth) {
  this.handle.style.marginLeft = timelineWidth + "px";
}
```
#### MouseDown and MouseUp
That's the hard part, but I'm not done yet. To make all of this work correctly, I've got to start by wiring up the MouseDown handler. Gotta click the handler for any of this to work. There's a trick to it though. If I were to add all 3 handlers directly to the handle, the MouseUp and MouseMove events would only fire if the mouse were to hover directly over the handle. Chances are, people aren't that disciplined to hold the mouse directly over the handle and will probably drag along the y-axis some. To solve this, the  MouseDown event  handler will add the MouseMove and the MouseUp events to the window.
```
mouseDown = (e) => {
  window.addEventListener('mousemove', this.mouseMove);
  window.addEventListener('mouseup', this.mouseUp);
};
```
And the MouseUp event handle will remove the handlers from the window.
```
mouseUp = (e) => {
  window.removeEventListener('mousemove', this.mouseMove);
  window.removeEventListener('mouseup', this.mouseUp);
};
```
Adding the MouseDown event handler to the handle is the final step to getting the timeline and handler to respond to mouse movements.
```
<div id="timeline" ref={(timeline) => { this.timeline = timeline }}>
  <div id="handle" onMouseDown={this.mouseDown} ref={(handle) => { this.handle = handle }} />
</div>
```
Done, and done!


