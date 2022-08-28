---

layout: post
title: "Understanding the 2D Camera in Raylib"
date:   2021-12-31 20:00:00 +0000
categories: tutorial

---

Welcome to my beginners tutorial for the 2D camera in the [raylib library](https://www.raylib.com/). For those of you unaware, raylib is a great, simple and easy to use C library for game development. 

Before we get going I want to lay my cards on the table. I am still very new to raylib and game development. I am writing this tutorial because I struggled with the 2d camera when first starting out. The API is very simple and easy to use but the concept was very new to me and it took me longer than I would have expected to get it. So the main purpose of this tutorial isn't to teach you how to code, but to help create an understanding of the 2d camera so you can use it effectively. 

If you would like to skip straight to some code, [here is the github gist](https://gist.github.com/ArchieAtkinson/5c5758ad68d5cfd55d40430ca8e9b44d).

## What is the camera trying to solve?
In a video game the world can range from a 2D pixel art dungeon in a rouge like to a realistic 3D war torn city in an FPS. It's the part of the game that the player sees and interacts with. A good way to see a game world is like a movie or TV sets instead of like the real world. They are crafted in order to trick the player into believing the world being shown is bigger, and more elaborate than it is. And in order to that effectively, like a movie or TV set, they use one or more camera's allowing different the switching to different perspectives easily. It can also be used to create a better game play experience, for example in most first person games when you get into a vehicle you are switched into a third person view to make driving easier. The way the camera acts in a game can make or break it so understanding how they work is an essential skill. This article will explore the most basic instance of using a camera and how we can implement it in raylib.

When first starting using raylib, you may have created a simple game where you have a box that acts as a player and you control it with the arrow keys. You quickly discover that you are confined to you screen. 

![](/assets/raylib-2dcamera/NoCamera.gif){:style="display:block; margin-left:auto; margin-right:auto"}

So now you want to be able to follow the box? That's where the camera comes in. 

## The 2D Camera in raylib
The a 2D camera in raylib is defined by a single structure `Camera2D` and managed by two functions, `BeginMode2D(Camera2D camera)` and `EndMode2D(void)`.

```c
// Camera2D, defines position/orientation in 2d space
typedef struct Camera2D {
    Vector2 offset;         // Camera offset (displacement from target)
    Vector2 target;         // Camera target (rotation and zoom origin)
    float rotation;         // Camera rotation in degrees
    float zoom;             // Camera zoom (scaling), should be 1.0f by default
} Camera2D;
```

To make use of a camera you need declared it and then assign values to each member of the `Camera2D` structure. Lets run through each member to see what it does. 

### Target

The target member is the camera's origin in the world. By default, when the offset member is {0, 0}, the camera starts in the top left corner of your screen and finished in the bottom right. Similar to how rectangles are drawn. So if your screen is 500px by 500px and you set the target to the position {-100, -200} you will see anything drawn in the world from {-100, -200} (top left corner) to {400, 300} (bottom right corner). 

![](/assets/raylib-2dcamera/Target.png){:style="display:block; margin-left:auto; margin-right:auto"}

Note: If you want this to follow a object like a player, this needs to be updated every time the player moves!

### Offset

The offset member is the origin of the camera on the screen. By default, it is in the top left corner. Changing the offset to `{SCREEN_WIDTH/2, SCREEN_HEIGHT/2}` moves the origin of the camera to the centre screen making the target now reference the centre instead of the top left corner. Using the example from above with the new offset, if the target was set to {-100, -200} you would now see anything drawn in the world from {-350, -450} (top left corner) to {150, 50} (bottom right corner). The target is now in the centre of screen.

![](/assets/raylib-2dcamera/Offset.png){:style="display:block; margin-left:auto; margin-right:auto"}

Note: Unlike target, once this is set, you only need to change it if you want to change the relative position of the object to the screen.

### Rotation and Zoom

The rotation and zoom members are is pretty self explanatory, just remember that they are centred around the target and that by default zoom equals one, not zero. 

### Putting it in action

With this new knowledge you can easy initialize a simple camera setup. 

```c
Camera2D camera;
camera.offset = (Vector2){SCREEN_WIDTH/2, SCREEN_HEIGHT/2};
camera.target = (Vector2){0, 0};
camera.rotation = 0.0f;
camera.zoom = 1.0f;
```
Now onto the functions. The two camera functions, `BeginMode2D(Camera2D camera)` and `EndMode2D(void)`. These will want to live in your main `draw()` function. You will always want to call `BeginMode2D(Camera2D camera)` first, and then draw some objects and then call `EndMode2D(void)`. 

```c
void draw(){
	BeginDrawing();
	ClearBackground(LIGHTGRAY);

	BeginMode2D(camera);

        draw_grid_background();
        
        draw_axis();

        draw_player(player);

	EndMode2D();

	EndDrawing();
}	
```

You can see I have indented in the `DrawRectangle()` and `draw_grid()` function. This is to make it clear that we are in "2D Mode" and making use of the camera. Now anything drawn between the two camera functions will be drawn with respect the world origin and not the screen origin. 

For example, using the structure and draw function we defined above, with a screen size of 500px by 500px, the displayed region of the world is {-250, -250} to {250, 250} with a red square in the centre at the world position {0, 0}. 

![](/assets/raylib-2dcamera/In2DMode.png){:style="display:block; margin-left:auto; margin-right:auto"}

If we moved the `DrawRectangle()` function to after the `EndMode2D()`. The red square would now be in the top left corner of the screen because that is the {0, 0} position of the screen. This will stay that way no matter where we set the camera target to as it's being drawn relative to screen, not the world. This isn't what we are looking for now but it's good to remember. It can be used to keep static element on the screen like a score tally, a minimap or a crosshair. The `GetWorldToScreen2D()` and the `GetScreenToWorld2D()` functions can help you move between the world and screen coordinates easily, useful for mouse interactions. For more information about these functions check out [raylib cheatsheet](https://www.raylib.com/cheatsheet/cheatsheet.html).

Now if we move `DrawRectangle()` function back in-between the 2d camera functions and add in some code to control the movement, like in the first gif, we end up with a red square that we can follow around the world. Perfect, just what we wanted! 

![](/assets/raylib-2dcamera/Camera.gif){:style="display:block; margin-left:auto; margin-right:auto"}

## Closing 

So what did we learn (I hope):
- Everything in-between the `BeginMode2D()` and `EndMode2D()` functions is draw relative to the world origin.
- Everything outside the `BeginMode2D()` and `EndMode2D()` functions is draw relative to the screen origin.
- The `target` member of the `Camera2D` structure sets where the origin of the camera is in the world.
- The `offset` member of the `Camera2D` structure sets where the origin of the camera is on the screen. 

There is a lot more to explore with the 2D camera (you still have zoom and rotate to play with!), and I would suggest you play around with the provided code to see what you can come up with. If you would like some inspiration, a Google doc entitled [Scroll Back - The Theory and Practice of Cameras in Side-Scrollers by Itay Keren](https://docs.google.com/document/d/1iNSQIyNpVGHeak6isbP6AHdHD50gs8MNXF1GCf08efg/pub) is a great showcase of different camera usage in side-scrollers.For some more complex examples, check out the 2D camera_platform example on the raylib git or [on the website](https://www.raylib.com/examples.html). 

The code used in this tutorial is on my [github as gist here](https://gist.github.com/ArchieAtkinson/5c5758ad68d5cfd55d40430ca8e9b44d).

I hope you found post this helpful. If you have any questions, comments or feedback feel free leave a comment on the [blogs github issue tracker](https://github.com/ArchieAtkinson/archieatkinson.github.io/issues/2) (I will work out proper commenting system one day...) or any of the links in the footer below. Thanks for reading!

