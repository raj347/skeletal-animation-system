skeletal-animation-system [![npm version](https://badge.fury.io/js/skeletal-animation-system.svg)](http://badge.fury.io/js/skeletal-animation-system) [![Build Status](https://travis-ci.org/chinedufn/skeletal-animation-system.svg?branch=master)](https://travis-ci.org/chinedufn/skeletal-animation-system)
===============

> A standalone, stateless, dual quaternion based skeletal animation system built with interactive applications in mind

[View live demo](http://chinedufn.github.io/skeletal-animation-system/)

## Background / Initial Motivation

skeletal-animation-system aims to give the user a flexible module for managing skeletal animations across different 3d models and bone groups.

`skeletal-animation-system` aims to provide a sane API for starting, stopping and interpolating skeletal animations.

It supports blending between
your previous and current animation when you switch animations. It also supports splitting your model into different bone groups such as the upper
and lower body, allowing you to, for example, play a walking animation for your legs while playing a punch animation for your upper body.

`skeletal-animation-system` does not maintain an internal state, but instead lets the modules consumer track thigns such as the current animation and the current clock time.

## Notes

This API is still experimental and will evolve as we use it and realize the kinks.

## To Install

```sh
$ npm install --save skeletal-animation-system
```

## Demo

To run the demo locally:

```sh
$ git clone https://github.com/chinedufn/skeletal-animation-system
$ cd skeletal-animation-system
$ npm install
$ npm run demo
```

Changes to the `demo` and `src` files will now live reload in your browser.

## Usage

```js
var animationSystem = require('skeletal-animation-system')
// Parsed using collada-dae-parser or some other parser
var parsedColladaModel = require('./parsed-collada-model.json')

// Animation names and the keyframes that they start and end on
// Zero indexed
//  ex: [2, 5] means start on the third keyframe and end at the sixth
var animationRanges = {
  'idle': [0, 5],
  'jump': [5, 8],
  'dance': [8, 14],
  'left-arm-punch': [14, 17],
  'right-arm-punch': [17, 20]
}

// Convert our joint names into their associated joint number
// This number comes from collada-dae-parser
// (or your compatible parser of choice)
var upperBodyJointNums = [
'spine', 'chest_L', 'chest_R', 'bicep_L', 'bicep_R'
].reduce(function (jointIndices, jointName) {
  jointIndices.push(parsedColladaModel.jointNamePositionIndex[jointName])
  return jointIndices
}, [])

var lowerBodyJointNums = [
'hip', 'thigh_L', 'thigh_R', 'femur_L'
].reduce(function (jointIndices, jointName) {
  jointIndices.push(parsedColladaModel.jointNamePositionIndex[jointName])
  return jointIndices
}, [])


var fullBodyJoints = upperBodyJoints.concat(lowerBodyJoints)

// Our options for animating our model's upper body
var upperBodyOptions = {
  currentTime: 28.24,
  keyframes: keyframes,
  jointNames: upperBodyJointsNums,
  blendFunction: function (dt) {
    // Blend animations linearly over 2.5 seconds
    return 1 / 2.5 * dt
  },
  currentAnimation: {
    range: animationRanges['left-arm-punch'],
    startTime: 25
  },
  previousAnimation: {
    range: animationRanges['right-arm-punch'],
    startTime: 24.5
  }
}

// Our options for animating our model's lower body
var lowerBodyOptions = {
  currentTime: 28.24,
  keyframes: keyframes,
  jointNums: lowerBodyJointNums,
  currentAnimation: {
    range: animationRanges['jump']
    startTime: 24.3
  }
}

var interpolatedUpperBodyJoints = animationSystem
.interpolateJoints(upperBodyOptions)

var interpolatedLowerBodyJoints = animationSystem
.interpolateJoints(lowerBodyJoints)

// Tou can pass these joint dual quaternions into `load-collada-dae`
var interpolatedJoints = Object.assign({}, interpolatedUpperBodyJoints, interpolatedLowerBodyJoints)
```

## Expected JSON model format

TODO: [Link to collada-dae-parser README]()

## TODO:

- [ ] expose bone groups names when exporting JSON from collada-dae-parser

## API

### `animationSystem.interpolateJoints(options)` -> `Object`

#### options

*Optional*

Type: `object`

```js
// Example overrides
var myOptions = {
  // TODO:
}
interpolatedJoints = animationSystem.interpolateJoints(myOptions)
```

##### currentTime

Type: `Number`

Default: `0`

The current number of seconds elapsed. If you have an animation an loop,
this will typically be the sum of all of your loops time deltas

```js
// Example of tracking current time
var currentTime = 0
function animationLoop (dt) {
 currentTime += dt
}
```

##### keyframes

Type: `Object`

Default: `{}`

TODO: Link to collada-dae-parser README on keyframes

##### jointNums

Type: `Array`

##### blendFunction

Type: `Function`

Default: `Blend linearly over 0.2 seconds`

A function that accepts a time elapsed in seconds
and returns a value between `0` and `1`.

This returned value represents the weight of the new animation.

```js
function myBlendFunction (dt) {
  // Blend the old animation into the new one linearly over 5 seconds
  return 0.2 * dt)
}
```

##### currentAnimation

Type: `Object`

An object containing parameters for the current animation

If you supply a previous animation your current animation will be
blended in using your `blendFunction`

```js
var currentAnimation = {
  range: [0, 10],
  startTime: 10
}
```

###### currentAnimation.range

Type: `Array`

The index of the first and last key for your current animation.

For example, if you have five keyframes at `0.2seconds`, `0.8s` `2.4s` `3.9s` and `13.8s`

A range of `[0, 2]` would loop between the keyframes at `0.2s` , `0.8s` and `2.4s`

A range of `[2, 3]` would loop between the keyframes at `2.4s` and `3.9s `

###### currentAnimation.startTime

Type: `Number`

The time in seconds that your current animation was initiated. This gets compared with
the `currentTime` in order to interpolate your joint data appropriately.

##### previousAnimation

An object containing parameters for the previous animations.
Your previous animation gets blended out using your `blendFunction`
while your current animation gets blended in.

Type: `Object`

###### previousAnimation.range

Type: `Array`

The index of the first and last key for your previous animation.

###### previousAnimation.startTime

Type: `Number`

The time in seconds that your previous animation was initiated.
This is used in order to blend in the current animation.

## See Also

- [load-collada-dae](https://github.com/chinedufn/load-collada-dae)
- [collada-dae-parser](https://github.com/chinedufn/collada-dae-parser)

## References

- [Anatomy of a skeletal animation system part 1](http://blog.demofox.org/2012/09/21/anatomy-of-a-skeletal-animation-system-part-1/), [part 2](http://blog.demofox.org/2012/09/21/anatomy-of-a-skeletal-animation-system-part-2/) and [part 3](http://blog.demofox.org/2012/09/21/anatomy-of-a-skeletal-animation-system-part-3/)

## License

MIT
