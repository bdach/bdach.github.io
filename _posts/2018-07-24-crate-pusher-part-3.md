---
layout: post
title: "Recreating Classic Video Games: Crate Pusher, Part 3"
sub_title: "Polishing up the core gameplay with animations."
image:
    path: "/assets/images/crate_pusher/hero.png"
    thumbnail: "/assets/images/crate_pusher/thumb.png"
categories: gamedev
comments: true
---

After a lengthy hiatus, caused in part by the exam period and job searching, CratePusher is back in full force!
In this installment I outline my efforts in polishing up the gameplay with some moderately flashy animations and transitions, using techniques such as input buffering, sprite cycling and animation curves.

# Animating the player sprite

## Following directions

## Sprite cycling

![Cycling just two player sprites can lead to acceptable results, for a small cost.](/assets/images/crate_pusher/player_animation.gif)

*Cycling just two player sprites can lead to acceptable results, for a small cost.*

# Adding fluidity to player actions

## Transitioning out of the discrete data model

## Animation duration and input buffering

# Improving level transitions --- yet another state machine

![The state machine used for scene transitions.](/assets/images/crate_pusher/scene_transition_machine.png)

*The state machine used for scene transitions.*

# Animation curves

<style type="text/css">
g.dotX {
    animation-name: movex;
    animation-duration: 3s;
    animation-iteration-count: infinite;
    transform-origin: 0.1px 0.1px;
}

g.dotY {
    animation-name: movey;
    animation-duration: 3s;
    animation-iteration-count: infinite;
    transform-origin: 0.1px 0.1px;
}

.linear {
    animation-timing-function: linear;
}

.cubic {
    animation-timing-function: cubic-bezier(.5, 0, .5, 1);
}

@keyframes movex {
    0%   { transform: translateX(-0.5px); }
    50%  { transform: translateX( 0.5px); }
    100% { transform: translateX(-0.5px); }
}

@keyframes movey {
    0%   { transform: translateY(-0.5px); }
    50%  { transform: translateY( 0.5px); }
    100% { transform: translateY(-0.5px); }
}
</style>

<svg viewBox="-0.6 -0.6 1.8 1.2"
     width="300"
     height="200"
     xmlns="http://www.w3.org/2000/svg">
    <path d="M -0.5 -0.5, 0.5 0.5"
          stroke="#90f6d799"
          stroke-width="0.05"
          fill="transparent" />
    <g class="dotX linear">
        <g class="dotY linear">
            <circle cx="0"
                    cy="0"
                    r="0.1"
                    fill="#90f6d7"
                    stroke="transparent" />
        </g>
    </g>
    <path d="M 1 -0.5 v 1"
          stroke="#90f6d799"
          stroke-width="0.05"
          fill="transparent" />
    <g class="dotY linear">
        <circle cx="1"
                cy="0"
                r="0.1"
                fill="#90f6d7"
                stroke="transparent" />
    </g>
</svg>

<svg viewBox="-0.6 -0.6 1.8 1.2"
     width="300"
     height="200"
     xmlns="http://www.w3.org/2000/svg">
    <path d="M -0.5 -0.5 C 0 -0.5, 0 0.5, 0.5 0.5"
          stroke="#90f6d799"
          stroke-width="0.05"
          fill="transparent" />
    <g class="dotX linear">
        <g class="dotY cubic">
            <circle cx="0"
                    cy="0"
                    r="0.1"
                    fill="#90f6d7"
                    stroke="transparent" />
        </g>
    </g>
    <path d="M 1 -0.5 v 1"
          stroke="#90f6d799"
          stroke-width="0.05"
          fill="transparent" />
    <g class="dotY cubic">
        <circle cx="1"
                cy="0"
                r="0.1"
                fill="#90f6d7"
                stroke="transparent" />
    </g>
</svg>

{% include youtubePlayer.html id="z585zO5KZMk" %}

# A bird's eye view on Crate Pusher

![The general hierarchy of components in Crate Pusher developed so far. Note the separation of concerns in the bottom-most layer.](/assets/images/crate_pusher/top_level_overview.png)

*The general hierarchy of components in Crate Pusher developed so far. Note the separation of concerns in the bottom-most layer.*

# What's next?
